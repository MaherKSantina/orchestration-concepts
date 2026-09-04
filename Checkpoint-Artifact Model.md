# API Documentation

## Overview

This model defines two core entities:

- `Artifact`
- `Checkpoint`

The system stores identity and historical change only. Aggregated or compiled state is derived later by replaying checkpoints.

Logical models such as product, design, code, proposals, or other domains are represented through artifact `kind` values and artifact-specific `data`. They do not require additional core storage models unless a system deliberately chooses to add them for implementation reasons.

---

## Artifact

### Description

An `Artifact` is a persistent entity introduced into the system once it has been discovered. It represents any modeled thing that needs identity and may be referenced or updated later.

### Fields

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `id` | `string` | No | Unique identifier of the artifact. |
| `kind` | `string` | Yes | Logical category of the artifact. |
| `data` | `json` | Yes | Artifact-specific JSON data belonging to the artifact itself. |
| `created_at` | `datetime` | No | Timestamp when the artifact was first introduced into the system. |

### Specification

- `id` must be unique across all artifacts.
- `kind` is optional and may be null when an artifact is discovered before it is fully understood.
- `data` stores intrinsic artifact information only.
- `Artifact` does not store aggregated or compiled state.
- `Artifact` should not store checkpoint references as canonical state. Historical causality is reconstructed from checkpoints.
- Artifact-to-artifact references are allowed when they describe domain structure, such as parent proposals, child proposals, dependencies, or target artifacts.

---

## Checkpoint

### Description

A `Checkpoint` is a flat historical record of an awareness-triggered event that may introduce new artifacts and apply changes to artifacts. A checkpoint may be produced manually or by executing an operation such as a function, rule, workflow, or import.

Checkpoints are the system's record of causality: they point to the artifacts they discovered or changed. Artifacts do not point back to checkpoints as part of canonical artifact state.

A checkpoint may be enriched after initial recording with additional context, labels, summaries, interpretations, or quality metadata. This keeps checkpoint history flat without creating checkpoint-about-checkpoint hierarchies. Enrichment should not change what the checkpoint originally produced.

### Fields

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `id` | `string` | No | Unique identifier of the checkpoint. |
| `created_at` | `datetime` | No | Timestamp when the checkpoint was recorded. |
| `updated_at` | `datetime` | Yes | Timestamp when the checkpoint record was last enriched or updated. |
| `trigger` | `json` | Yes | Raw description of the event or signal that triggered awareness. |
| `context` | `json` | Yes | Contextual information available during the checkpoint. |
| `discovered` | `json` | No | List of artifacts first introduced by the checkpoint, each containing an `id` and optional initial artifact data. |
| `operation` | `json` | Yes | Description of the mechanism that produced the checkpoint changes, such as a manual action, function execution, rule, workflow, or import. |
| `changed` | `json` | No | List of artifact changes produced by the checkpoint. |
| `enrichment` | `json` | Yes | Later-added context, labels, summaries, interpretations, review notes, or quality metadata about the checkpoint. |
| `capture` | `json` | Yes | Metadata describing how completely the `changed` set was recorded, especially when technical constraints prevent full inline storage. |

### Specification

- `id` must be unique across all checkpoints.
- `discovered` introduces artifact existence.
- `changed` records modifications to artifacts.
- The same artifact may appear in both `discovered` and `changed` in the same checkpoint.
- `operation` records how the changes were produced, not the changes themselves.
- `context` records contextual information available during the checkpoint event.
- `enrichment` records information added after the checkpoint event to improve interpretation or review.
- Checkpoint history remains flat: avoid modeling checkpoint-about-checkpoint hierarchies.
- If more information is learned about a checkpoint, update or enrich the checkpoint record itself rather than creating a new checkpoint whose subject is the previous checkpoint.
- Aggregated artifact state is computed later by replaying checkpoints in order.
- `capture` must make incompleteness explicit whenever the full `changed` set is not stored inline.
- To keep replay stable, event-producing fields such as `id`, `created_at`, `trigger`, `operation`, `discovered`, and `changed` should be treated as stable after creation.
- Later updates should generally be limited to enrichment, annotation, review, labeling, capture metadata, or other non-replay-producing information.

### Operation Shape

The `operation` field is intended to support both manual and executable change production.

Example structure:

```json
{
  "type": "function",
  "name": "normalize_entities",
  "params": [
    0.8,
    "use only artifacts discovered in the current checkpoint"
  ],
  "description": "Normalize artifact naming and merge equivalent entities"
}
```

Recommended semantics:

- `type`: mechanism category such as `manual`, `function`, `rule`, `workflow`, or `import`
- `name`: identifier of the function, rule, workflow, or action when applicable
- `params`: ordered parameter array containing absolute values and/or natural-language inputs
- `description`: optional human-readable explanation of what was executed

### Changed Shape

The `changed` field represents the resulting artifact-level effects of the checkpoint.

Example structure:

```json
[
  {
    "artifact_id": "artifact_123",
    "change_type": "update",
    "data": {
      "title": "Refined title",
      "status": "validated"
    }
  }
]
```

Recommended semantics:

- Each entry should identify the affected artifact.
- Each entry should describe the resulting modification.
- `changed` should describe result state or patch intent, not execution metadata.

### Capture Shape

The `capture` field exists to describe how much of the `changed` set was retained.

Example structure:

```json
{
  "mode": "externalized",
  "total_changed": 120430,
  "captured_changed": 500,
  "reason": "checkpoint exceeded inline storage threshold",
  "ref": "checkpoint_changes/batch_2026_04_02_01"
}
```

Recommended modes:

- `full`: all changed items stored inline
- `externalized`: full changed set stored outside the main checkpoint record, with a reference
- `sampled`: only representative changed items stored inline
- `summary_only`: aggregate metadata stored without full changed items
- `dropped`: changed items not stored, with explicit reason


---

## Relationship Direction

The model uses one primary direction for historical causality:

```text
Checkpoint -> Artifact
```

A checkpoint references artifacts through `discovered` and `changed`. An artifact does not need to reference the checkpoints that created, reviewed, approved, rejected, or applied it.

Historical questions are answered by querying checkpoints:

```text
Find checkpoints where:
  discovered contains artifact_id
  OR changed contains artifact_id
```

Recommended relationship guidance:

- Use artifact-to-artifact references for domain structure.
- Use checkpoint-to-artifact references for historical causality.
- Avoid artifact-to-checkpoint references in canonical artifact state.
- Avoid checkpoint-to-checkpoint hierarchies; enrich the relevant checkpoint instead.

Examples of valid artifact-to-artifact references:

```json
{
  "parent_proposal_id": "proposal_001",
  "child_proposal_ids": [
    "proposal_001_product",
    "proposal_001_design",
    "proposal_001_code"
  ],
  "depends_on": [
    "proposal_001_product"
  ],
  "target_artifacts": [
    "artifact_checkout_flow"
  ]
}
```

---

## Proposed Change Modeling

A proposed change is modeled as an `Artifact`, not as a checkpoint.

The proposal artifact stores the current modeled proposal state, such as title, status, rationale, target artifacts, proposed effects, dependencies, and child proposals. The checkpoint records the event that introduced, expanded, reviewed, approved, rejected, or applied that proposal.

Example proposal artifact:

```json
{
  "id": "proposal_001",
  "kind": "proposed_change",
  "data": {
    "title": "Redesign checkout flow",
    "status": "draft",
    "target_artifacts": [
      "artifact_checkout_flow"
    ],
    "proposed_changed": [
      {
        "artifact_id": "artifact_checkout_flow",
        "change_type": "update",
        "data": {
          "steps": ["cart", "shipping", "payment", "review"]
        }
      }
    ]
  },
  "created_at": "2026-05-07T01:00:00Z"
}
```

Example checkpoint that creates the proposal:

```json
{
  "id": "checkpoint_100",
  "created_at": "2026-05-07T01:00:00Z",
  "trigger": {
    "type": "user_proposed_change"
  },
  "context": {
    "source": "planning_session"
  },
  "discovered": [
    {
      "id": "proposal_001",
      "kind": "proposed_change"
    }
  ],
  "operation": {
    "type": "manual",
    "name": "create_proposal"
  },
  "changed": [
    {
      "artifact_id": "proposal_001",
      "change_type": "update",
      "data": {
        "title": "Redesign checkout flow",
        "status": "draft"
      }
    }
  ],
  "capture": {
    "mode": "full"
  }
}
```

Recommended rule:

```text
Proposal details -> Artifact.data
Proposal relationships -> Artifact.data
Proposal status -> Artifact.data
Proposal creation/review/approval/application events -> Checkpoints
```

---

## Cascaded Proposed Changes

A proposal may cascade into multiple related proposals across dimensions such as product, design, code, documentation, or operations without introducing new core models.

Each proposal remains an `Artifact` with `kind: "proposed_change"`. The cascade is represented through artifact-to-artifact links in `data`.

Example parent proposal:

```json
{
  "id": "proposal_001",
  "kind": "proposed_change",
  "data": {
    "role": "parent",
    "title": "Redesign checkout flow",
    "status": "expanded",
    "child_proposal_ids": [
      "proposal_001_product",
      "proposal_001_design",
      "proposal_001_code"
    ]
  },
  "created_at": "2026-05-07T01:00:00Z"
}
```

Example child proposal:

```json
{
  "id": "proposal_001_design",
  "kind": "proposed_change",
  "data": {
    "role": "child",
    "dimension": "design",
    "parent_proposal_id": "proposal_001",
    "status": "draft",
    "target_artifacts": [
      "artifact_checkout_wireframes"
    ],
    "depends_on": [
      "proposal_001_product"
    ],
    "proposed_changed": [
      {
        "artifact_id": "artifact_checkout_wireframes",
        "change_type": "update",
        "data": {
          "layout": "four-step checkout"
        }
      }
    ]
  },
  "created_at": "2026-05-07T01:05:00Z"
}
```

Example checkpoint that records the cascade:

```json
{
  "id": "checkpoint_110",
  "created_at": "2026-05-07T01:05:00Z",
  "trigger": {
    "type": "proposal_cascade_requested",
    "proposal_id": "proposal_001"
  },
  "context": {
    "dimensions": ["product", "design", "code"]
  },
  "discovered": [
    {
      "id": "proposal_001_product",
      "kind": "proposed_change"
    },
    {
      "id": "proposal_001_design",
      "kind": "proposed_change"
    },
    {
      "id": "proposal_001_code",
      "kind": "proposed_change"
    }
  ],
  "operation": {
    "type": "workflow",
    "name": "cascade_proposal",
    "params": [
      "proposal_001",
      ["product", "design", "code"]
    ],
    "description": "Derived dimension-specific child proposals from the parent proposal."
  },
  "changed": [
    {
      "artifact_id": "proposal_001",
      "change_type": "update",
      "data": {
        "status": "expanded",
        "child_proposal_ids": [
          "proposal_001_product",
          "proposal_001_design",
          "proposal_001_code"
        ]
      }
    }
  ],
  "capture": {
    "mode": "full"
  }
}
```

Recommended rule:

```text
Cascaded proposals are linked through artifacts.
The cascade event is recorded by a checkpoint.
Applied effects are recorded by later checkpoints.
```

---

## Checkpoint Enrichment and Flat History

To avoid checkpoint hierarchy, the system may update a checkpoint record with later information instead of creating checkpoints about checkpoints.

This means a checkpoint is the current best record of a historical event. It contains both event-time information and later enrichment.

Recommended structure:

```json
{
  "id": "checkpoint_110",
  "created_at": "2026-05-07T01:05:00Z",
  "updated_at": "2026-05-07T03:00:00Z",
  "trigger": {
    "type": "proposal_cascade_requested",
    "proposal_id": "proposal_001"
  },
  "context": {
    "dimensions": ["product", "design", "code"]
  },
  "operation": {
    "type": "workflow",
    "name": "cascade_proposal"
  },
  "discovered": [],
  "changed": [],
  "enrichment": {
    "summary": "Generated product, design, and code child proposals from proposal_001.",
    "labels": ["proposal", "cascade", "cross_dimension"],
    "interpretation": {
      "type": "proposal_expansion",
      "models_involved": ["product", "design", "code"]
    },
    "review": {
      "status": "reviewed",
      "notes": "Cascade structure is valid."
    }
  },
  "capture": {
    "mode": "full"
  }
}
```

Guidance:

- Keep checkpoint history flat.
- Do not create checkpoint-about-checkpoint hierarchies by default.
- Add later understanding to `enrichment` on the checkpoint itself.
- Avoid changing replay-producing fields after creation.
- If a replay-producing field must be corrected, treat it as a correction of the record and make that explicit in `enrichment` or operational metadata.

---

## Replay and Derived Views

Artifacts do not need to store their own history. Artifact history is derived from checkpoints.

To reconstruct the current state of an artifact:

```text
1. Find the checkpoint that discovered the artifact.
2. Replay checkpoints in created_at order.
3. Apply entries in changed where artifact_id matches the artifact.
4. Use the chosen capture strategy to include inline or externalized changes.
```

To reconstruct proposal history:

```text
Find checkpoints where:
  discovered contains proposal_id
  OR changed contains artifact_id = proposal_id
```

To reconstruct a cascaded proposal graph:

```text
1. Replay proposal artifacts.
2. Read artifact-to-artifact links such as parent_proposal_id, child_proposal_ids, depends_on, and target_artifacts.
3. Query checkpoints only when historical causality or event context is needed.
```

## Storage Specification

### Artifacts

| Column | Type | Nullable |
|---|---|---:|
| `id` | `string` | No |
| `kind` | `string` | Yes |
| `data` | `json` | Yes |
| `created_at` | `datetime` | No |

### Checkpoints

| Column | Type | Nullable |
|---|---|---:|
| `id` | `string` | No |
| `created_at` | `datetime` | No |
| `updated_at` | `datetime` | Yes |
| `trigger_json` | `json` | Yes |
| `context_json` | `json` | Yes |
| `discovered_json` | `json` | No |
| `operation_json` | `json` | Yes |
| `changed_json` | `json` | No |
| `enrichment_json` | `json` | Yes |
| `capture_json` | `json` | Yes |

### Recommended Scalable Storage Layout

For systems where a checkpoint may affect a very large number of artifacts, the logical checkpoint model should remain the same while physical storage may be split for scale.

#### checkpoints

| Column | Type | Nullable |
|---|---|---:|
| `id` | `string` | No |
| `created_at` | `datetime` | No |
| `updated_at` | `datetime` | Yes |
| `trigger_json` | `json` | Yes |
| `context_json` | `json` | Yes |
| `discovered_json` | `json` | No |
| `operation_json` | `json` | Yes |
| `enrichment_json` | `json` | Yes |
| `capture_json` | `json` | Yes |
| `changed_preview_json` | `json` | Yes |

#### checkpoint_changes

| Column | Type | Nullable |
|---|---|---:|
| `id` | `string` | No |
| `checkpoint_id` | `string` | No |
| `artifact_id` | `string` | No |
| `change_json` | `json` | No |

### Storage Guidance

- The logical model still treats `changed` as part of the checkpoint.
- Physical storage may externalize large change sets for performance and indexing.
- `capture` should describe whether `changed` is fully inline, partially inline, summarized, or stored externally.
- `enrichment_json` may be updated after checkpoint creation to keep checkpoint history flat without introducing checkpoint hierarchy.
- Replay-producing fields should be treated as stable so historical reconstruction remains explicit and correct.
- Replaying checkpoints must account for the chosen storage strategy so historical reconstruction remains explicit and correct.
