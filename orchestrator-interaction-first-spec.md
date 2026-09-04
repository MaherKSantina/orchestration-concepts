# Orchestrator Interaction-First Product Specification

## Purpose

Define the business rules, use cases, and UI model for an interaction-first orchestration system.

This system starts from real or simulated interactions, externalizes recurring structures, and promotes them into explicit process definitions. The process graph remains the canonical orchestration artifact once formalized, but it is not the first artifact created.

---

## Product Thesis

The product should mirror how work happens in real life:

1. An interaction begins.
2. Information is exchanged through governed requests and responses.
3. Sub-requests appear while resolving the work.
4. Repeated structures become visible.
5. Those structures are externalized into an explicit process.
6. The process becomes the analyzable, reusable orchestration artifact.

This preserves the distinction between:

- **origin artifact**: interaction trace
- **canonical orchestration artifact**: explicit process definition

---

## Core Principles

### 1. Interaction-first, process-later
The first captured artifact is the interaction, not the process graph.

### 2. Process remains canonical after formalization
Once externalized, the process definition is the durable artifact for storage, execution, analysis, governance, and reuse.

### 3. Graph and interaction serve different purposes
- **Interaction model** captures how work unfolded.
- **Graph model** captures the stable structure of the process.

### 4. A step is not the same thing as an interaction
- A **step** is a durable unit of work in a formalized process.
- An **interaction** is a runtime exchange that may later reveal a step boundary.

### 5. Roles matter more than literal people
The system should model interactions between contextual parties in roles, not only between user accounts.

### 6. Discovery precedes orchestration
The system should support moving from implicit work patterns to explicit orchestration artifacts.

### 7. Real and hypothetical work must remain distinct
Live, observed, internal, and simulated interactions may share the same structure, but they must not be mixed analytically unless explicitly intended.

---

## System Layers

### 1. Discovery layer
Captures interactions as they happen or are reconstructed.

### 2. Pattern layer
Groups repeated interactions into candidate structures.

### 3. Process draft layer
Produces an initial explicit graph from one or more candidates.

### 4. Orchestration layer
Stores the formal process definition and its graph topology.

### 5. Execution layer
Runs steps locally using human, agent, tool, or system actors.

---

## Primary Objects

## Interaction
A bounded exchange between two contextual parties.

An interaction:
- starts with a request from the source party
- may contain requests for more information
- may spawn child interactions recursively
- concludes when the destination provides the final response for that interaction

An interaction is the first-class object of discovery.

## Message
A structured event inside an interaction.

A message is not free-form chat as a system primitive. It is a governed event that contributes to analyzable work progression.

## Party
A contextual participant in an interaction.

A party is not only a person. It may represent:
- a human in a role
- the same human in a different role
- an agent
- a tool
- an external actor
- a self-role used for internal delegation

## Candidate
A discovered recurring interaction shape that may deserve formalization.

## Process Draft
A generated or manually assembled graph proposal derived from one or more candidates.

## Process Definition
The explicit, bounded orchestration artifact represented as steps, edges, bindings, and actors.

---

## Party Model

The system should separate:

- **identity**: who or what the participant is
- **role**: what function they are performing in this interaction
- **mode**: whether the interaction is live, internal, observed, or simulated
- **channel**: where the interaction came from

This allows one human to appear multiple times in a workflow without collapsing distinct functions.

### Examples
- Strategist You -> Executor You
- Executor You -> Analyst You
- You -> Vendor
- Vendor -> You
- Prospect (simulated) -> You

---

## Interaction Modes

### Live
A real interaction that actually happened.

Examples:
- chat
- email
- tool-to-user request
- user-to-agent exchange

### Internal
A self-handoff between roles embodied by the same person.

Examples:
- strategist delegates to executor
- analyst reviews output from executor
- planner asks reviewer to assess quality

### Observed
A reconstructed interaction based on a different communication surface.

Examples:
- a phone call summarized into structured requests and responses
- a meeting discussion normalized into interaction events
- another person’s behavior modeled from notes

### Simulated
A hypothetical interaction used for planning or scenario analysis.

Examples:
- what to say if the customer objects
- what happens if a manager asks for proof
- what to do if a vendor requests an additional document

### Rule
Simulated interactions must be visually and analytically distinct from real interactions.

---

## Channel Types

The interaction model must support multiple source channels.

Minimum channel types:
- chat
- email
- call
- meeting
- manual entry
- tool interaction
- scenario

A channel does not change the structure of the interaction model. It only changes the source medium and presentation metadata.

---

## Message Model

Each interaction contains a sequence of structured messages or events.

### Minimum message types
- `request_key`
- `provide_value`
- `final_response`
- `note`
- `observation`
- `inference`

### Message requirements
A request should support:
- a key
- optional context
- optional expected shape
- optional required flag

This avoids over-compressing semantics into short labels and improves future analysis.

### Example
Instead of only:
- `customer_email`

Prefer:
- key: `customer_email`
- context: `needed to retrieve account`
- expected_shape: `email`
- required: `true`

---

## Interaction State Rules

Each interaction should have an explicit state.

Recommended states:
- `open`
- `awaiting_source`
- `awaiting_destination`
- `concluded`
- `cancelled` later if needed

This is separate from process and step status.

---

## Child Interaction Rules

A party may request additional information while resolving a parent interaction.

When that happens:
- a child interaction may be created
- the child interaction resolves the requested missing information
- the child result returns to the parent as the answer to the missing request
- the parent interaction may then continue or conclude

### Rule
Child interactions are recursive sub-exchanges, not automatically process steps.

### Rule
A recurring child interaction pattern is a candidate for future step or subprocess promotion.

---

## Externalization Rules

### Rule 1
Interactions are the source material for discovery.

### Rule 2
A process should only be created when a stable recurring structure is visible.

### Rule 3
A candidate may be formed from one interaction or many similar interactions.

### Rule 4
Promotion to a process should preserve lineage back to source interactions.

### Rule 5
The graph is the formalization surface, not the initial capture surface.

### Rule 6
The graph should remain the explicit topology once formalized.

### Rule 7
Local exchanges that explain how a step reached its output should remain inspectable from the step.

---

## Relationship Between Interactions and Steps

### Before formalization
The primary object is the interaction.

### After formalization
The primary orchestration object is the step.

### Runtime rule
A step instance may contain one or more interaction threads that explain how the step output was produced.

### Modeling rule
Do not assume every interaction maps one-to-one to a future step.

Possible outcomes:
- several interactions collapse into one step
- one interaction becomes multiple steps
- one child interaction becomes a subprocess
- an interaction remains only evidence and is never promoted

---

## UI Model

## Product Modes

### 1. Discovery Mode
Default mode in early-stage use.

Purpose:
- capture work as interaction
- inspect recursion
- identify repeated patterns
- externalize hidden structure

Primary objects:
- interactions
- messages
- child interactions
- extracted keys
- recurring requests

### 2. Process Mode
Formalization mode for explicit orchestration.

Purpose:
- convert candidates into explicit topology
- inspect and refine graph structure
- bind actors, inputs, outputs, and routing

Primary objects:
- process definitions
- steps
- edges
- bindings
- actors

---

## Primary Screens

## 1. Interaction Inbox
List of interactions across all modes and channels.

Each row should show:
- title or summary
- mode
- channel
- source role
- destination role
- current state
- final outcome if concluded
- child interaction count
- unresolved request count
- candidate grouping if any

Purpose:
- start from real work
- browse and filter interactions
- identify likely process candidates

## 2. Interaction Detail
The core discovery screen.

Should show:
- full thread timeline
- nested child interactions
- mode and channel badges
- from-role and to-role labels
- requested keys
- provided values
- inferred outputs
- unresolved asks
- summary of final outcome
- similar interactions panel
- candidate formation actions

Purpose:
- understand how the work unfolded
- extract hidden step boundaries
- identify repeated missing inputs

## 3. Candidate Patterns
A grouped view of recurring structures.

Each candidate should show:
- common initiating request
- common roles involved
- common requested keys
- common child interactions
- common final outcomes
- example interactions
- promotion readiness

Purpose:
- bridge discovery and formalization

## 4. Process Draft Builder
The first graph surface.

Should support:
- seeded step suggestions from candidate structure
- actor assignment
- edge review
- input/output contract shaping
- manual merging or splitting of candidate steps

Purpose:
- transform candidate patterns into an explicit graph

## 5. Process Graph / Process Detail
The explicit orchestration view using xyflow.

Each node should surface:
- step name
- actor
- status in runs
- interaction count
- unresolved request count
- output presence

Selecting a node should open a step inspector.

## 6. Step Inspector
Tabs should include:
- overview
- bindings
- input/output
- interactions
- insights

Purpose:
- connect explicit orchestration back to runtime evidence

---

## xyflow Representation Rules

### Rule 1
xyflow represents explicit process topology, not raw interaction history.

### Rule 2
Interactions should be attached to steps for inspection, not shown as graph nodes by default.

### Rule 3
Child interactions should appear as nested interaction structures in the step inspector or interaction detail view.

### Rule 4
The graph may be seeded from interaction candidates, but once formalized it becomes its own explicit artifact.

---

## Business Rules

## Interaction Rules
1. An interaction involves exactly two parties for MVP.
2. The first message is always a request from the source party.
3. The destination may either:
   - conclude the interaction with a final response, or
   - request more information.
4. A request for more information may create a child interaction.
5. When a child interaction concludes, its final response may satisfy the parent request.
6. An interaction is concluded only when the destination produces the final response for that interaction.

## Role Rules
1. A single person may appear in multiple roles.
2. Role transitions are valid interaction boundaries even when identity does not change.
3. Internal self-handoffs are first-class interactions.
4. Role labels must be explicit in the UI.

## Simulation Rules
1. Simulated interactions must be clearly labeled.
2. Simulated interactions must not affect real analytics by default.
3. Simulated branches may be used for planning, rehearsal, and strategy.
4. A simulated interaction may later inform a real process, but only through explicit promotion.

## Observation Rules
1. Non-chat communication may be normalized into structured interactions.
2. Observed interactions should preserve source channel metadata.
3. Inferred requests and responses should support confidence metadata when useful.

## Process Promotion Rules
1. Not every interaction should become a process.
2. Promotion requires evidence of recurring structure or strategic value.
3. Promotion should preserve lineage to source interaction examples.
4. A promoted process becomes the canonical orchestration artifact.

## Runtime Rules
1. Step instances may store the resolved input and final output.
2. Step instances may also reference the interactions that produced the output.
3. Waiting states should be representable at the interaction level immediately.
4. Process-level and step-level waiting and failure states should be added soon after MVP if human execution is important.

---

## MVP Scope

### In scope
- interaction inbox
- interaction detail with nested child interactions
- role-aware parties
- mode labels: live, internal, observed, simulated
- multi-channel metadata
- candidate grouping
- process draft creation from candidates
- xyflow process view
- step inspector with linked interactions
- recurring requested key analysis

### Out of scope for day 1
- graph editing through message threads
- multi-party interactions
- automatic process promotion without review
- fully free-form chat as a canonical model
- deep AI-based semantic normalization as a dependency for core usability
- raw interaction nodes on the main graph by default

---

## Data Model Implications

## Existing Orchestration Model
The current orchestration model already supports:
- process definitions
- actors
- step definitions
- context bindings
- edges
- process instances
- step instances

These remain the core of the formalized orchestration layer.

## Recommended Discovery Additions

### `interaction_threads`
Represents a bounded interaction.

Suggested fields:
- `id`
- `mode`
- `channel_type`
- `source_party_id`
- `destination_party_id`
- `state`
- `parent_interaction_id` nullable
- `source_reference` nullable
- `summary` nullable
- `created_at`
- `updated_at`

### `interaction_messages`
Represents a structured event within an interaction.

Suggested fields:
- `id`
- `interaction_thread_id`
- `sender_party_id`
- `receiver_party_id`
- `message_type`
- `key` nullable
- `payload` nullable
- `context` nullable
- `expected_shape` nullable
- `is_required` nullable
- `sequence_number`
- `parent_message_id` nullable
- `child_interaction_id` nullable
- `confidence` nullable
- `created_at`

### `parties`
Represents contextual participants.

Suggested fields:
- `id`
- `identity_id` nullable
- `actor_id` nullable
- `role_name`
- `party_type`
- `display_name`
- `created_at`
- `updated_at`

### `interaction_candidates`
Represents recurring patterns that may deserve promotion.

Suggested fields:
- `id`
- `name`
- `summary`
- `status`
- `derived_from_query` or clustering reference
- `created_at`
- `updated_at`

### `interaction_candidate_examples`
Links exemplar interactions to candidates.

### `process_lineage_links`
Links processes and possibly steps back to source candidates or exemplar interactions.

---

## Status Model Recommendation

The current orchestration schema is intentionally minimal, but it will likely need expansion soon.

Recommended additions after MVP:
- failed process state
- failed step state
- waiting-for-human process state
- waiting-for-human step state
- blocked interaction state if useful

The interaction model will surface these needs early because waiting and incomplete information are central to discovery.

---

## Use Cases

## 1. Real cross-party work execution
A person requests work from another person or agent. The destination requests more information, receives it, and concludes the task.

Value:
- captures how work really happened
- reveals recurring missing inputs
- creates evidence for future formalization

## 2. Internal self-delegation
One person operates in multiple roles and hands work to themselves.

Example:
- strategist defines an approach
- executor performs low-level work
- analyst reviews results
- planner decides next move

Value:
- externalizes hidden mental workflows
- identifies step boundaries inside solo work

## 3. Cross-channel normalization
A call, meeting, or external exchange is captured as a structured interaction.

Value:
- unifies work traces across mediums
- allows analysis independent of communication surface

## 4. Modeling another person’s behavior
A user records what another person is asking for, providing, or withholding.

Value:
- makes partner behavior analyzable
- supports preparation and pattern recognition

## 5. What-if scenario planning
A user simulates possible replies and response branches before a real interaction happens.

Value:
- supports preparation and strategic rehearsal
- enables branch visualization without contaminating live analytics

## 6. Process discovery from recurring work
The system groups similar interactions and reveals common keys, roles, and child-interaction shapes.

Value:
- identifies where structure already exists informally
- creates natural process candidates

## 7. Process externalization and formalization
One or more candidates are promoted into an explicit graph.

Value:
- turns informal recurring behavior into reusable orchestration
- preserves analyzability and future automation potential

## 8. Step-level runtime audit
A formal process run allows a user to inspect the exact interactions that led a step to produce its output.

Value:
- preserves traceability
- explains output formation
- improves governance and debugging

---

## Promotion Signals

A candidate is more likely to deserve promotion when one or more of the following is true:
- the same roles appear repeatedly
- the same requested keys appear repeatedly
- the same child interaction structure repeats
- the same outcome is produced repeatedly
- the same delay or blockage appears repeatedly
- the work is strategically important even if low-volume

---

## Analytics Opportunities

The interaction-first model should eventually support:
- most-requested keys by candidate or process
- most common missing inputs
- most common child interactions
- most common role transitions
- average recursion depth
- common successful resolution paths
- process promotion suggestions

Simulated interactions should be excluded by default from these analytics unless explicitly included.

---

## Product Positioning

Orchestrator is an interaction-first process externalization system.

It starts from real, internal, observed, or simulated work interactions, normalizes them into governed exchange structures, discovers recurring patterns, and promotes them into explicit process graphs that can be analyzed, executed, and improved.

---

## Summary Decision

The recommended product direction is:

- **Interactions are the origin of process discovery.**
- **Candidates are the bridge from behavior to structure.**
- **Processes are the formalized orchestration artifact.**
- **xyflow is the formalization and inspection surface, not the initial capture surface.**
- **Roles, modes, and channels are essential dimensions of the discovery model.**
- **Child interactions reveal missing information paths and future subprocess boundaries.**

