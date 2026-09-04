# Methodology Machinery
## Component reference with instantiations for code and specs as source of truth

The methodology has been described concretely so far — annotations on code, lifting from code to spec, AI miss interrogation on code edits. This document extracts the machinery from those concrete descriptions so it can be applied to any artifact serving as the source of truth, not just code.

Two instantiations are shown side by side near the end: code-as-source-of-truth (the original framing) and spec-as-source-of-truth (the SDD-compatible inversion). The machinery is identical across both; only the artifact it operates on differs.

---

## Why extract the machinery

The earlier framing implicitly bound the methodology to a specific source of truth (code). Three things become visible once the binding is loosened.

First, the methodology is *artifact-agnostic*. It doesn't presuppose that code is canonical or that specs are derived; it presupposes only that *something* is canonical and that other artifacts are described against it. Either side of the SDD debate can adopt the machinery without buying the other side's substrate.

Second, multiple instantiations can coexist on the same codebase. A system can have some regions where code is the source of truth (the methodology operates on code, with specs as annotations) and other regions where specs are the source of truth (the methodology operates on specs, with code as the derived artifact). The per-region choice is independent of the methodology.

Third, the polarity dimensions from the framework (coverage stance, decision classification, methodology evolution) are orthogonal to the source-of-truth choice. You can have either source-of-truth allocation with either an open or closed coverage stance, either uniform or tiered classification, and either designed or emergent evolution. The machinery doesn't predetermine any of them.

---

## Core components

### Entities

**Source artifact.** The artifact that is authoritative for some concern. Direct edits to it are the primary change action. Other artifacts are derived from it via generation or describe it via annotation. The source artifact's identity defines the instantiation.

**Derived artifact.** A thing produced from a source artifact by a transformation that may be deterministic (a generator, a compiler, a template) or stochastic (AI-mediated). Edits to a derived artifact do not propagate to the source automatically; they're either lost on regeneration or treated as exceptions.

**Annotation.** Intent, invariant, or contextual information attached to a source artifact via a stable anchor. Captures what the source artifact itself cannot express. Annotations split into two layers:
- **Floor annotation**: intent and context that no AI could derive from the source artifact alone. Examples: business rationale, "we tried X and it failed," regulatory constraints, invariants that span multiple sources, anti-patterns the team has learned to avoid.
- **Overlay annotation**: model-specific compensation. Things the current AI keeps getting wrong about this source, which a more capable AI might not need.

**Anchor.** A stable semantic identifier that attaches an annotation to a region of the source artifact. Anchors must survive refactoring — they reference named entities, behavioral signatures, or controlled identifiers rather than text positions.

**Test (or behavioral signature).** A mechanical verification of expected behavior. Tests are the equivalence gate for lifting: a region cannot be promoted from source-of-truth status to derived-from-something-higher status without tests demonstrating that the new generation produces behaviorally equivalent output.

**Generator.** A transformation that emits a derived artifact from a source artifact (and possibly its annotations). Generators are deterministic when the relationship can be made formal (compilation, structured templates) or AI-mediated when the relationship is underdetermined (most prose-to-implementation flows).

**Validation surface.** The artifact against which output correctness is observed — where tests run and where behavioral inspection happens. In some instantiations the validation surface is the same artifact as the source (you edit code, you validate code). In others it is a derived artifact downstream of the source (you edit specs, you validate the generated code). When the source-to-output chain has more than one generation step — for example when lifting has created an intermediate abstraction layer between the original source and the final output — each step's output is a candidate *intermediate validation surface*. The *primary* validation surface is the most concrete artifact, where final behavior is observed (typically tests on running code). Intermediate validation surfaces are diagnostic: when the primary surface reports a miss, you can validate at each intermediate to localize which generator in the chain produced the wrong output. The relationship between source and validation surface shapes the miss-to-annotation loop: when they coincide, the loop is single-layer; when they diverge, the loop requires a routing step to direct annotations to the right layer, and intermediate surfaces help with that routing by localizing the miss before it gets attributed.

### Operations

**Edit.** Change the source artifact directly. The primary mutation; the unit of change for unannotated and annotated regions.

**Annotate.** Attach an annotation to a region via an anchor. Annotation is cheap; most observations stop at annotate rather than escalating to lift.

**Lift.** Promote a region from "edited directly" to "generated from a higher artifact." Lifting changes which artifact is the source of truth for that region. Requires a generator and an equivalence test to license the change. Lifting is the operation that increases determinism in the system.

**Demote / inline.** The reverse of lift. Rare. Happens when a lifted abstraction never varies and the abstraction's overhead exceeds the savings it provides.

**Generate.** Run a generator to produce derived artifacts from a source.

**Detect drift.** Mechanical check that the source artifact at an anchor matches the annotation's expectation — signature match, behavioral test pass, checksum match. Drift is a deterministic signal that doesn't require AI judgment.

**Interrogate miss.** When AI's output differs materially from intent, ask AI to articulate its assumption, then verify the stated assumption by checking whether an annotation based on it prevents the miss on the next attempt. The interrogation produces an annotation candidate; the verification confirms or rejects it.

**Prune.** Remove overlay annotations that observation shows are no longer needed (e.g., a more capable model now handles the case without the annotation). Pruning is the reverse of annotating and runs on the overlay layer only.

### Signals

**AI miss.** AI's output diverged from intent enough that material human correction was needed. A miss is a signal that the source artifact or its annotations didn't carry enough information for the AI to do the right thing.

**Churn.** Observed rate at which a specific decision is re-litigated over time. Decision-scoped, not region-scoped.

**Reach.** Fan-out of consequences when a decision changes. The number of distinct places that must adapt or are constrained by the change.

**Experimentation tempo.** Anticipated rate of variation. Forward-looking churn — used when historical churn data doesn't yet exist.

**Drift indicator.** Mechanical mismatch between annotation expectation and anchor state.

**Coverage ratio.** Fraction of changes that stay in the structured layer (lifted or annotated) versus those that fall through to the escape hatch. The success metric for the methodology overall.

### Classifications

**Region status:**
- **Unannotated**: source artifact region with no attached intent capture. AI operates on it with source-only context.
- **Annotated**: source artifact region with annotations attached. AI operates with source plus intent context.
- **Lifted**: region whose source-of-truth has migrated to a higher artifact; the original source artifact is now derived for this region.

**Annotation layer:**
- **Floor**: model-permanent intent and constraints.
- **Overlay**: model-specific compensation.

**Decision quadrant (churn × reach):**
- **High churn, high reach → Lift now**: first-class structured element in the higher artifact.
- **High churn, low reach → Lift lightly**: local parameter or knob.
- **Low churn, high reach → Freeze, watch**: stays in source, but instrumented for churn changes.
- **Low churn, low reach → Leave**: stays in source as settled sediment.

### Feedback loops

**Miss-to-annotation loop.** A miss is observed on the validation surface (failed test, incorrect behavior, material human correction required) and produces an annotation candidate, but the path from observation to annotation has two branches.

The first branch depends on whether the transformation that produced the miss was AI-mediated or deterministic. For AI-mediated transformations (unlifted regions where AI interpreted source into output), interrogate AI for the stated assumption that produced the miss, then verify the assumption by checking whether an annotation based on it prevents the miss on the next attempt. For deterministic transformations (lifted regions where a template or generator ran without AI judgment), there's no AI to interrogate; the miss has to trace back to the structured input or the template definition, and the fix lands there.

The second branch depends on whether the validation surface and annotation surface are the same artifact. When they coincide (you edit and test the same artifact), the resulting annotation lands on the source itself in one move. When they differ (you edit a source artifact but validate a derived one), the annotation candidate requires routing: source-local clarification (the source was ambiguous), source-adjacent annotation layer (project conventions were missing), or both (a cross-cutting invariant that should be captured at both levels).

The keep-or-discard step is the same regardless of branch: keep the annotation if the next attempt verifies it, revise or remove it if the same shape of miss recurs.

**Churn-to-lift loop.** Observed churn × reach crosses threshold for a region → propose lift → write generator → equivalence-test against existing behavior → cut over so the region becomes generated rather than directly edited.

**Overlay-prune loop.** New model behavior shows an overlay annotation is no longer needed (the model handles the case unaided) → retire the annotation (or tag as legacy for fallback compatibility).

**Drift-repair loop.** Mechanical drift detected at anchor → repair via one of: re-pin the anchor (the locus moved due to refactoring), revise the annotation (the intent itself changed), or update the source (the drift is a bug to fix).

---

## How the parts play together

A change request enters the system. The operating sequence:

1. **Locate** the relevant region(s) in the source artifact via the annotation graph or direct navigation.
2. **Classify** the region(s) as unannotated, annotated, or lifted.
3. **Discover reach** through annotation cross-references — the annotations themselves are the reach map.
4. **Apply the change** at the unit appropriate to the classification:
   - **Lifted region**: edit the higher artifact, regenerate, equivalence-test.
   - **Annotated region**: edit source artifact with annotation context (AI-assisted or manual).
   - **Unannotated region**: edit source artifact with source-only context.
5. **Record** what was touched, what shape the edit had, and how AI behaved (if AI was involved).
6. **Run the feedback loops** triggered by the resulting signals:
   - Miss detected → miss-to-annotation loop.
   - Churn × reach signal crossed → churn-to-lift loop.
   - Overlay redundancy observed → overlay-prune loop.
   - Anchor drift detected → drift-repair loop.

Two structural invariants hold the system together regardless of instantiation:

**Role purity.** Each artifact is wholly source-of-truth or wholly derived for any given concern. The boundary moves region by region as lifting and demotion happen, but at any moment, no artifact straddles both roles within a region. When a mixed entity appears, it must be split into two role-pure entities joined on a shared key.

**One-way generation.** Cascade flows from higher artifacts to lower. Edits at lower artifacts are captured locally (as escape-hatch contributions) or treated as drift; they do not propagate upward automatically because the upward translation is generally non-invertible.

These two invariants are what make the machinery internally consistent across instantiations. Any instantiation that violates either of them breaks the methodology.

---

## Instantiation A: code as source of truth

In this instantiation, code is what gets edited directly, what's authoritative for behavior, and what tests are verified against. The spec layer, when it exists, lives as annotations on code.

| Component | Realized as |
|---|---|
| **Source artifact** | Code (source files in the repository) |
| **Derived artifact (downward)** | Assembly / bytecode (via compilation) |
| **Derived artifact (upward, when lifted)** | Generated code emitted from spec functions |
| **Annotation** | Tech specs, product specs, README sections attached to code regions |
| **Anchor** | Function names, class names, named exports, behavioral signatures |
| **Test** | Behavioral test suite (unit, integration, e2e) |
| **Generator (downward)** | The compiler |
| **Generator (upward, when lifted)** | Custom spec-to-code generators built per lifted region |
| **AI miss** | AI's code edit required material human correction |
| **Drift indicator** | Code at an anchor has different signature or behavior than its annotation expects |
| **Floor annotation example** | "This routine must be O(log n); we tried O(n) and it failed under production load" |
| **Overlay annotation example** | "Current AI uses async/await syntax where this codebase prefers .then() chaining" |
| **Lift target** | A spec function or template that generates this region's code when invoked with structured parameters |
| **Validation surface** | Same as source artifact — code is what gets edited and code is what tests run against |

**Operating loop in this instantiation.** Tickets arrive; you locate the code region; classify (most code unannotated by default in a fresh codebase); edit (manually or AI-assisted with whatever annotations exist for the region); record AI behavior. AI misses produce annotation candidates that get verified on subsequent attempts. Recurring same-shape edits across many code regions produce lift candidates — patterns that should become spec functions emitting their code. Lifting means writing the spec function, equivalence-testing the generated output against the existing code's behavior, and cutting over so that region's code becomes generated rather than hand-edited. Because the validation surface (code) is the same artifact as the primary annotation surface (code with tech-spec annotations), the miss-to-annotation loop runs in one step — no routing decision is required, and interrogation always applies because AI is the transformation for unlifted regions.

**Default coverage stance is open.** Most code stays as source-of-truth, unlifted; lifted regions are the exception, earned by observation. The escape hatch (hand-editing code) is the default mode, not the exception.

---

## Instantiation B: spec as source of truth

In this instantiation, the spec is what gets edited directly, what's authoritative for behavior, and what code is generated against. Code becomes the derived artifact; the instruction layer (AGENTS.md, skills documents) plays the role annotations played in Instantiation A.

| Component | Realized as |
|---|---|
| **Source artifact** | Spec documents (PRD, technical spec, structured spec elements) |
| **Derived artifact (downward)** | Code (generated by AI implementation or deterministic template) |
| **Derived artifact (further downward)** | Assembly / bytecode (via compilation of the generated code) |
| **Annotation** | Instruction layer (AGENTS.md, skills documents, area-specific guides) |
| **Anchor** | Spec section headers, named requirements, identified clauses |
| **Test** | Behavioral test suite (often derived from the spec's verification criteria) run against the generated code |
| **Generator (lift-level, spec-to-spec)** | Deterministic generator that turns higher-level structured spec elements into more concrete spec elements. Created when a region of the spec is lifted. The output is spec, not code. |
| **Generator (bottom-level, spec-to-code)** | At the bottom of the spec layer: AI implementation pipeline when the concrete spec is prose; deterministic template when the concrete spec is itself structured |
| **AI miss** | The chain produced incorrect code somewhere along the path from source spec to running code; localization may require checking intermediate validation surfaces |
| **Drift indicator** | Spec section changed but dependent sections weren't updated; or a generator's output diverged from what its tests or downstream consumers expect |
| **Floor annotation example** | "Compliance requires this section to mention encryption at rest"; "We tried prose for the auth flow and the AI kept misinterpreting it — this section must stay as structured fields" |
| **Overlay annotation example** | "Current AI reads 'should' as 'must' in policy language; clarify with explicit MUST/MAY when encountered" |
| **Lift target** | A higher-level structured spec construct that generates more concrete spec elements. Lifting in B produces spec, not code — code generation is a separate downstream step at the bottom of the spec layer. |
| **Primary validation surface** | The generated code — tests run on the most concrete derived artifact |
| **Intermediate validation surfaces** | When lifting has created multiple spec abstractions, each spec layer is a candidate validation point: you can verify that a lift generator produced the right concrete spec before checking whether the code derived from that spec is correct |

**Operating loop in this instantiation.** Tickets arrive; you locate the spec section relevant to the change; classify (most spec sections are prose with AI-mediated code generation; some have been lifted into multi-layer structured forms with deterministic spec-to-spec generators between abstraction levels; some are unannotated prose with no instruction-layer context); edit at the appropriate abstraction level; the generator chain produces concrete spec and then code; verify against tests, using intermediate validation surfaces to localize any misses to a specific generator in the chain. AI misses produce instruction-layer candidates (additions to AGENTS.md or area skills). Recurring same-shape spec sections produce lift candidates — patterns that should migrate from prose to higher-level structured forms with their own deterministic generators. Lifting in B creates a spec-to-spec generator, not a spec-to-code generator: the lift output is spec at a more concrete level, and code generation happens separately at the bottom of the spec layer (where it can itself be deterministic if the bottom spec is structured, or AI-mediated if the bottom spec is prose).

Because validation runs on the derived artifact (code) but annotations attach to the source artifact (spec) or its annotation layer (instructions), the miss-to-annotation loop has a routing step. The first move when the chain is more than one step long is *localization*: check intermediate validation surfaces to identify which generator in the chain produced the miss. If a lift-level spec-to-spec generator output the wrong concrete spec, the miss localizes there (fix the higher-level spec or the generator definition). If the lift-level output was correct but the bottom-level spec-to-code step went wrong, the miss is downstream (fix the bottom spec, its template, or the AI's interpretation). Once localized, the routing rules apply: if the miss was caused by genuine ambiguity in the spec at the relevant level, the annotation refines the spec itself; if it was caused by a project convention the AI didn't know, the annotation goes in the instruction layer (AGENTS.md, skills); if it came from something AI fundamentally couldn't infer (a regulatory constraint, an anti-pattern history, a cross-spec invariant), it's floor annotation territory and the layer choice depends on scope. In deterministic regions of the chain (lift-level generators, structured-spec-to-code templates), there's no AI to interrogate — the fix lands in the structured input or the template definition, not in any annotation layer.

**Coverage stance is independent.** The default for SDD-aligned environments is closed (no code edits without spec changes), but the machinery itself doesn't require it. An open variant allows hand-editing code as an escape hatch, with subsequent reverse-flow that updates the spec or instruction layer to capture what was learned.

---

## Instantiation C: agent conversations as source of truth

In this instantiation, the source artifact is the running stream of messages between agents in a multi-agent system, accumulating across sessions. There is no derived artifact downstream — messages are terminal in the chain. Annotations layer interpretive structure over the message stream to make large session histories digestible. Lifting transforms AI-mediated agent behavior into deterministic handlers for recurring interaction patterns.

| Component | Realized as |
|---|---|
| **Source artifact** | Messages between agents, accumulating across sessions (append-only — past messages cannot be edited, only new ones produced) |
| **Derived artifact** | None downstream — messages are terminal |
| **Annotation** | Interpretive overlays on message regions: gists, strategies, overarching direction, summaries that make session history navigable and that may feed back into agent instructions |
| **Anchor** | Specific messages or message ranges identified by stable IDs (highly stable — messages are immutable once produced) |
| **Test** | Outcome-based: did the conversation reach resolution, satisfy task requirements, maintain policy adherence, hit metric targets, avoid escalation |
| **Generator (when not lifted)** | AI agent producing messages from session context |
| **Generator (when lifted)** | Deterministic handler taking structured inputs and emitting messages or actions without AI mediation |
| **AI miss** | An agent produced an inappropriate message — wrong tone, wrong content, missed context, policy violation, wrong tool invocation |
| **Drift indicator** | A pattern annotated as "strategy X" no longer reflected in current messages; or a lifted handler's outputs diverging from the outcomes its exemplars produced |
| **Floor annotation example** | "These agents always end customer conversations with a status summary"; "Compliance requires policy violations to escalate to human review, never resolve within the agent loop" |
| **Overlay annotation example** | "Current model misreads brevity as dissatisfaction; flag short responses as deliberate"; "Current model over-apologizes; trim apologetic phrasing in summaries" |
| **Lift target** | A deterministic agent handler replacing an AI-mediated interaction pattern — takes structured inputs parsed from conversation context, emits outputs without AI generation in that loop |
| **Validation surface (pre-lift, for unlifted regions)** | Same as source — messages are what gets observed and assessed |
| **Validation surface (post-lift, for that region)** | The messages produced by the handler — observable on the message stream, but now produced from structured inputs rather than AI free generation |

**Operating loop in this instantiation.** Sessions run; messages accumulate as agents handle interactions; you sample sessions (or use automated analysis) to identify regions worth annotating; classify (most message streams are unannotated runtime behavior; some interactions have been lifted to deterministic handlers; some have annotations capturing strategy or direction). When something needs to change — an agent behaved wrongly, an outcome was bad — you locate the relevant messages, interrogate the producing agent's reasoning if AI-mediated, and update either the agent's persistent instructions (the annotation analog: rules in agent system prompts or session context that influence future behavior), or lift the interaction into a deterministic handler if the pattern recurs enough to justify it. Verification happens by running new sessions and observing whether the issue recurs on the message stream itself.

Lifting in C produces a deterministic handler — a structured replacement for the AI agent within a specific interaction pattern. The handler takes structured inputs (parsed from the conversation context) and produces structured outputs (messages or actions). Pre-lift, an interaction is AI-mediated end to end; post-lift, the lifted segment runs deterministically while the rest of the conversation may still flow through AI. The pattern matches A and B: the previously-source artifact (messages from free AI generation) becomes the output of a generator that produces it deterministically from structured input.

**Two structural features distinguish C from A and B.** First, the source is *append-only* — you cannot edit past messages; you can only add new ones. The "edit" operation in C effectively becomes "change the agent's behavior so future messages differ," which is closer to changing a generator than mutating an artifact. The underlying source artifact only grows; revision happens upstream of production rather than on the produced artifact itself. Second, C is *dual-mode for the validation surface*: pre-lift, source = validation surface (you observe messages directly to assess correctness), so the miss-to-annotation loop is symmetric like A and runs in one step; post-lift for a given region, source moves up to "structured inputs to the handler" while validation stays on the produced messages, so the loop becomes asymmetric for that region like B. C inherits A-style simplicity in unlifted regions and B-style routing in lifted regions, both at once.

A third feature worth naming: annotations in C are heavily *interpretive* (read-after-the-fact understanding) rather than *declarative* (intent stated up front). In A and B, you typically annotate by declaring what code or spec is meant to do. In C, annotations tend to extract structure from observed runtime behavior — patterns surfaced from many sessions. The same annotation slot serves two purposes that are more separable in A and B: making past sessions navigable for human review, and feeding back into agent instructions to shape future behavior. Both purposes coexist; the emphasis on after-the-fact interpretation is what distinguishes C.

---

## Cross-instantiation observations

A few things become visible only when the instantiations sit side by side.

**Lifting produces a generator for the previous source artifact.** In Instantiation A, the previous source was code, so the lift generator produces code (with structured spec input). In Instantiation B, the previous source was spec, so the lift generator produces concrete spec (with higher-level structured input); code generation is a separate step at the bottom of the spec layer, independent of lifting. In Instantiation C, the previous source was messages, so the lift generator (a deterministic agent handler) produces messages (with structured input parsed from conversation context). The "what does lifting output" question always answers with the artifact that used to be authoritative — lifting moves the source up, and the new source generates what the old source was. All three instantiations reduce variation in the relationship between input and what-used-to-be-the-source; the cost is up-front structural work, the benefit is predictable downstream production of the previously-source artifact.

**Validation surface relationship to source varies across instantiations.** In A, source and validation surface coincide pre-lift (you edit code and test code), and diverge for lifted regions (spec is source, code is validation). In B they always diverge (spec is source, code is validation), with chain length growing as lifting adds intermediate spec layers. In C, source and validation surface coincide pre-lift (messages are both edited-by-production and observed), and diverge for lifted regions like A does. The miss-to-annotation loop's complexity follows: symmetric in A's unlifted and C's unlifted regions (single-layer); routing-required in B always, in A's lifted regions, and in C's lifted regions. The pattern is that the loop becomes asymmetric whenever lifting introduces a layer between source-of-truth and the place where output is observed.

**Chain length affects how much intermediate validation matters.** In Instantiation A, the chain from source to validation surface is typically short — code is both edited and tested, and lifting adds one intermediate step (spec function → code). In Instantiation B the chain is longer by default (spec → code → tests on code) and lifting extends it further by inserting spec-to-spec generators between abstraction levels (high-level spec → mid-level spec → bottom spec → code). In Instantiation C the chain is minimal pre-lift (messages observed directly), and adds one step per lifted region (structured input → handler → message). As the chain lengthens, intermediate validation becomes more valuable because there are more places a miss can originate. Localizing the miss before routing the annotation prevents attributing it to the wrong layer. This is one reason operating in B has higher overhead than A or C — not just routing, but localization across multiple potential miss sources.

**The "annotation layer" is the same thing under different names.** What's called a spec in Instantiation A is what's called an instruction layer in Instantiation B is what's called persistent agent instructions (system prompts, session context, behavior rules) in Instantiation C. All three capture what the source artifact can't express on its own. AGENTS.md, tech-spec annotations on code, and agent system prompts exist for the same reason: AI needs context the source artifact doesn't provide, and that context has to live somewhere addressable.

**Floor and overlay translate symmetrically across all three instantiations.** Floor annotations are durable intent that no AI could derive from the source; overlay annotations are model-specific compensation. The split is independent of which artifact is canonical or whether the source is static or runtime.

**The escape hatch differs across instantiations.** In Instantiation A, code edits *are* the primary mode, so "escape hatch" is just normal operation. In Instantiation B, code edits are escape from the spec layer, and whether they're allowed becomes a coverage-stance policy decision — this is where SDD adopters typically experience the most operational friction. In Instantiation C, the escape hatch is human intervention in a running conversation (a human takes over from an agent, or hand-crafts an outgoing message), which is similarly friction-laden because it desynchronizes future agent behavior from the observed conversation history.

**Source artifacts can be static or runtime.** A and B operate on statically-edited artifacts (code files, spec documents) that mutate in place. C operates on a runtime-generated artifact (the message stream) that grows append-only. This affects what "edit" means in each: in static instantiations, edit mutates the artifact directly; in runtime instantiations, edit changes the producer of future artifact additions (the agent, its instructions, or a lifted handler). The annotation and lift mechanisms work the same way, but the implicit mutation model differs and operators have to internalize it.

**Annotations can be declarative or interpretive.** In A and B, annotations are typically declarative — you state intent ahead of operation ("this code does X because Y"). In C, annotations tend to be interpretive — extracting meaning from observed runtime behavior ("these sessions show pattern Z"). The same annotation slot serves both purposes in any instantiation, but the dominant mode shifts based on whether the source is static (intent precedes artifact, so declarative dominates) or runtime (artifact precedes interpretation, so interpretive dominates). Recognizing which mode is dominant helps with how the annotation work is scheduled: declarative annotations are made up-front during design; interpretive ones happen during periodic review of accumulated runtime behavior.

**Multiple instantiations on one system are supported.** A system can have some regions in Instantiation A (annotated code, lifted selectively), other regions in Instantiation B (spec-driven, code generated), and others in Instantiation C (agent conversations with deterministic handlers lifted out where patterns recur). The per-region choice depends on which artifact is most naturally authoritative for that concern. UI-heavy features with stable patterns often fit B; algorithmic core with high variation often fits A; agent-mediated workflows with repeating interaction shapes fit C.

**Recursion stops at one layer.** The temptation to apply the machinery to the methodology document itself, or to AGENTS.md as if it were yet another source artifact, produces diminishing returns. One inversion is enough. The methodology document is a meta-artifact describing how to run the methodology; the methodology's working state lives in the instantiations, not in the document. Layering further annotations on the document produces philosophy rather than productivity.

---

## What the abstraction reveals

Several useful consequences emerge from having the machinery extracted.

The choice of source-of-truth is a deployment decision, not a methodology decision. The same machinery works in either instantiation; switching between them is a matter of remapping components rather than changing the methodology. This dissolves the apparent conflict between code-canonical and spec-canonical frames: both are valid deployments of the same underlying machinery.

The methodology composes cleanly with SDD on the source-of-truth dimension. By accepting Instantiation B as your operational mode, you adopt SDD's source-of-truth position (Dimension 1 of the framework) while keeping the rest of your methodology's machinery intact. The polarity dimensions (especially coverage stance) remain open negotiation points but are independent of the instantiation choice.

The machinery's components are individually adoptable. A team that doesn't want the full methodology can take pieces: just the floor-versus-overlay distinction for their AGENTS.md, just the miss-interrogation loop, just the churn × reach quadrant for spec sections. The components don't lose value when separated from the whole — they just provide less coverage.

The methodology and the methodology document are different things. The document is a description; the methodology is what's actually running in the instantiations. Confusing them produces over-investment in the document and under-investment in the running practice. The document earns its keep only when changes to it are driven by changes in what's running, not the reverse.
