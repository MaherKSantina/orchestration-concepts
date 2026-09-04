# Left System Functional Specification
## Formalized Process-Orchestration System (FPOS)

## 1. Purpose

This document specifies a **left system**: a process-centered orchestration environment for defining, executing, exploring, evaluating, and evolving work.

The system is intended to support two modes of reality at once:
- **formalized work**, where the process is already known and should be executed reliably
- **emerging work**, where the process is not yet stable and must be explored, compared, and progressively formalized

The left system is not defined by any single tool category such as workflow automation, BPMN modeling, agent orchestration, or process mining. It combines aspects of all of them into one coherent system whose main purpose is to help a human rapidly develop, test, and evolve process logic without being blocked by missing orchestration features.

The system must support deterministic tools, human tasks, AI tasks, child processes, discovery flows, and formal execution flows under a single conceptual model.

---

## 2. System Goal

The system shall allow a user to:
- define a process explicitly
- execute that process repeatedly
- inspect every step and all produced artifacts
- compare alternative process paths
- test process stability and boundaries
- evolve the process without breaking historical instances
- spawn and manage child processes recursively
- use AI as a bounded step capability rather than as the sole driver of the whole system

The outcome is a process operating environment where the user can move quickly without rebuilding orchestration fundamentals midstream.

---

## 3. Core Design Principles

### 3.1 Process as a first-class object
The fundamental unit of the system is the **process**, not the ticket, prompt, file, or API call.

### 3.2 Orchestration as a first-class object
Routing, branching, retries, pauses, resumes, joins, and failure handling are explicit system concepts, not hidden implementation detail.

### 3.3 Progressive formalization
Unknown work should be explored first and only promoted into formal reusable process definitions when evidence is strong enough.

### 3.4 Auditability by default
Every execution must be inspectable. Inputs, outputs, decisions, retries, branch choices, and produced artifacts must be reconstructable.

### 3.5 AI compatibility without AI dependence
AI-capable steps are supported natively, but AI must fit the same process contracts, evidence requirements, and inspection model as every other step type.

### 3.6 Stable definitions and isolated execution
Process definitions must evolve safely while historical and in-flight process instances remain interpretable and runnable.

### 3.7 Hierarchical composition
A process may spawn child processes. Parent-child relationships must be explicit in both definition and runtime views.

---

## 4. In Scope

The system shall support:
- visual process graph editing
- formal process execution
- exploratory process modeling
- orchestration control flow
- process instances and runtime inspection
- retries and replay
- comparison of alternative process paths
- analytics and process path aggregation
- process versioning
- child process orchestration
- mocking and simulation
- import/export of process definitions
- optional natural-language-assisted authoring

---

## 5. Out of Scope

This specification does not define:
- storage engines
- user interface frameworks
- deployment model
- authentication or infrastructure details
- external vendor APIs
- model-specific prompting syntax

Those choices belong to implementation, not functional behavior.

---

## 6. Core Object Model

This section defines the minimum conceptual objects required for the system to function.

### 6.1 Process Definition
A reusable description of how work is structured.

A Process Definition must contain:
- unique identity
- name and summary
- process type
- declared goal
- input contract
- output contract
- step graph
- routing rules
- evaluation rules
- escalation rules
- child-process declarations
- supported execution modes
- version metadata
- status such as draft, active, deprecated, archived

A Process Definition changes when:
- process logic is updated
- a step is added, removed, or changed
- contracts change
- routing behavior changes
- evaluation logic changes
- child-process relationships change

### 6.2 Process Version
An immutable snapshot of a Process Definition at a point in time.

A Process Version exists so that:
- historical instances remain interpretable
- in-flight instances keep their original semantics unless explicitly migrated
- changes can be reviewed and diffed
- backward compatibility can be enforced

A new Process Version is created when a change would alter execution meaning.

### 6.3 Process Instance
A single runtime execution of a specific Process Version.

A Process Instance must contain:
- identity
- reference to Process Version
- initiation cause
- instance state
- current position in the graph
- runtime input values
- produced outputs
- runtime variables
- audit timeline
- linked artifacts
- linked child instances
- evaluation outcomes
- retry history
- operator comments and overrides

A Process Instance changes when:
- execution advances
- values are produced or updated
- a step retries
- a branch is chosen
- a child process starts or completes
- a human intervenes
- evaluation marks the instance as passed, failed, blocked, or escalated

### 6.4 Step Definition
A single node within a Process Definition.

Step types shall include at minimum:
- deterministic action
- human task
- AI task
- child-process invocation
- router / condition
- merge / convergence
- wait / pause
- evaluator
- mock / simulation step
- artifact transform

Each Step Definition must declare:
- step identity
- step type
- input contract
- output contract
- dependencies
- side-effect policy
- retry policy
- timeout policy
- idempotency expectations
- mockability
- observability requirements

### 6.5 Step Attempt
A concrete execution attempt of a Step Definition inside a Process Instance.

A Step Attempt must capture:
- identity
- referenced step
- attempt number
- start and end timestamps
- consumed input values
- produced outputs
- status
- error details if any
- operator intervention if any
- evaluation notes if any

A new Step Attempt is created when:
- a step runs the first time
- the step is retried
- the step is replayed from a checkpoint
- the step is rerun under modified conditions

### 6.6 Contract
A formal description of what a process or step expects and what it promises.

Contract types shall include:
- input contract
- output contract
- evidence contract
- evaluation contract
- side-effect contract

Contracts exist to:
- make execution comparable
- make steps interchangeable
- constrain AI and human tasks the same way
- support versioning and backward compatibility

### 6.7 Artifact
A named result produced or consumed by a process or step.

Examples include:
- plan
- code diff
- test result
- screenshot
- decision record
- evaluation report
- metrics bundle
- theory comparison report

An Artifact must contain:
- identity
- artifact type
- producer reference
- input lineage
- content payload
- schema / shape identifier
- timestamp
- confidence or validation state where applicable

Artifacts change when:
- a new revision is produced
- evaluation or human review changes its status
- downstream processes annotate it

### 6.8 Evaluator
A reusable rule or procedure that determines whether a step or process output is acceptable.

An Evaluator may be:
- rule-based
- threshold-based
- comparison-based
- human review-based
- AI-assisted but bounded
- composite

An Evaluator must record:
- evaluation target
- expected conditions
- observed conditions
- pass/fail/inconclusive result
- evidence references
- rationale

### 6.9 Theory
A candidate explanation or process hypothesis used in exploratory work.

A Theory must contain:
- statement of hypothesis
- expected value or benefit
- uncertainty level
- target process or component
- planned tests
- success indicators
- risk level
- status such as proposed, active-test, promoted, rejected

A Theory changes when:
- evidence is gathered
- confidence changes
- test results arrive
- it is promoted into a formal process element

### 6.10 Boundary Test
A structured exploration used to find where a process, step, component, or theory holds or fails.

A Boundary Test must capture:
- target component or process
- manipulated variables
- tested range or conditions
- observed outcomes
- discovered limits
- failure patterns
- recommendation

### 6.11 Audit Event
A timestamped, append-only record of meaningful runtime activity.

Audit Events shall cover:
- instance created
- step started
- step completed
- retry scheduled
- retry performed
- branch selected
- child process started
- child process completed
- pause / resume
- human intervention
- evaluator result recorded
- process cancelled
- process failed
- process completed
- process migrated or forked

### 6.12 Snapshot / Checkpoint
A restorable runtime state of a Process Instance or subgraph.

Checkpoints exist to support:
- pause and resume
- retry from a known safe point
- branch exploration
- forensic replay
- debugging and comparison

---

## 7. Process Types

The system shall support at least three top-level process types.

### 7.1 Execution Process
Used when the path is known enough to run reliably.

Primary purpose:
- perform work
- produce outputs
- enforce contracts
- support routine operation

### 7.2 Discovery Process
Used when the path is uncertain and alternatives must be explored.

Primary purpose:
- compare theories
- test candidate steps or routes
- quantify uncertainty reduction
- identify promising process shapes

### 7.3 Consolidation Process
Used to decide whether exploratory learning should be promoted into reusable structure.

Primary purpose:
- analyze evidence from discovery
- choose what becomes formal process logic
- retire weak theories
- propose new stable versions

These process types may call each other recursively.

---

## 8. Major Use Cases and Required Behavior

## 8.1 Define a formal process

The system shall allow a user to visually create or edit a process that is already understood.

The system must read:
- process metadata
- existing steps and edges
- contracts
- evaluator bindings
- version history
- reusable subprocess library

The system must save:
- process graph
- contracts per process and per step
- orchestration rules
- process version metadata
- authoring comments and rationale

Information changes because:
- the process is refined
- a stable discovery becomes formalized
- a broken path is repaired
- contracts are tightened or relaxed
- routing logic is improved

## 8.2 Explore an emerging process

The system shall allow a user to represent work that is not yet formalized and treat it as a measurable exploratory object.

The system must support:
- theory creation
- candidate graph variants
- partial or provisional steps
- uncertain contracts
- explicit unknowns
- side-by-side comparison of variants
- quantitative and qualitative result capture

The system must read:
- theories
- prior experiments
- candidate process fragments
- metrics and outcomes
- recorded edge cases

The system must save:
- exploratory graph versions
- theory records
- experiment plans
- comparative outcomes
- promotion recommendations

Information changes because:
- new evidence arrives
- candidate paths are rejected or refined
- theories gain or lose confidence
- a path becomes stable enough to promote

## 8.3 Manipulate steps and test stability

The system shall allow a user to alter steps, parameters, routes, or dependencies in order to find robust process candidates.

The system must support:
- enabling/disabling steps
- swapping alternative steps
- changing route conditions
- adjusting retries and timeouts
- rerunning from checkpoints
- branching from historical runs
- mocking dependencies
- stress or edge-case execution

The system must read:
- process topology
- checkpoint state
- historical runtime data
- evaluator results
- component capabilities

The system must save:
- scenario definitions
- branch experiments
- forked run records
- result deltas
- stability findings

Information changes because:
- manipulated runs produce different outcomes
- the user introduces a new candidate component
- failures reveal hidden boundaries

## 8.4 Orchestrate complex execution flows

The system shall treat orchestration as a first-class capability.

The system must support:
- sequencing
- conditional routing
- branching
- convergence
- parallel execution
- retries
- pause and resume
- cancellation
- compensation / rollback hooks
- scheduled re-entry
- import/export of definitions
- mocking and dry runs
- process diffs

The system must read:
- runtime conditions
- route predicates
- dependency completion states
- retry policies
- branch join conditions
- import/export payloads

The system must save:
- routing decisions
- active branches
- branch joins
- retry history
- cancellation reasons
- process definition diffs
- mock run outcomes

Information changes because:
- state changes at runtime
- an external signal arrives
- a branch condition is resolved
- a failure forces compensation or retry

## 8.5 Inspect and audit runtime instances

The system shall allow a user to inspect a running or completed instance deeply.

The system must provide:
- instance timeline
- current graph position
- step-level values
- artifacts by step
- branch history
- retry history
- child process links
- evaluator outcomes
- override history
- checkpoint history

The system must read:
- runtime state
- step attempts
- artifacts
- audit events
- child instance relations

The system must save:
- annotations
- bookmarks
- operator notes
- manually attached evidence
- correction and override records

Information changes because:
- execution advances
- reviews are recorded
- operators annotate or intervene

## 8.6 Use AI-capable steps without special-case architecture

The system shall treat AI as a first-class step capability, not as an external exception to the process model.

AI steps must behave like other steps in that they declare:
- input contract
- output contract
- side-effect policy
- evaluator binding
- retry policy
- confidence or uncertainty capture
- artifact outputs

The system must support AI-specific metadata where relevant, such as:
- prompt or instruction artifact
- context bundle artifact
- model capability label
- inference result artifact
- optional confidence or explanation fields

The system must read:
- structured input
- context artifacts
- capability constraints
- evaluator expectations

The system must save:
- generated outputs
- consumed context references
- AI-specific artifacts
- run metadata
- review and correction history

Information changes because:
- prompts or context inputs evolve
- model behavior differs across retries or versions
- downstream evaluation rejects or approves results

## 8.7 Analyze process behavior over time

The system shall produce aggregated process intelligence, not only single-run inspection.

The system must support analysis of:
- common paths
- rare paths
- edge cases
- repeated failure points
- retry-heavy areas
- unstable branches
- evaluator failure patterns
- common child-process cascades
- time to completion by path
- theory success rates

The system must read:
- historical process instances
- runtime paths
- step attempts
- boundary test outcomes
- artifacts and evaluations

The system must save:
- aggregated metrics
- path summaries
- anomaly detections
- discovered edge-case clusters
- stability scores
- candidate optimization recommendations

Information changes because:
- additional instances are executed
- versions change behavior
- new edge cases emerge

## 8.8 Evolve process definitions safely

The system shall support process versioning and backward-compatible change management.

The system must support:
- immutable versions
- version comparison and diff
- activation / deprecation lifecycle
- compatibility validation
- explicit migration policy
- coexistence of multiple active versions
- stable interpretation of historical instances

The system must read:
- current and historical definitions
- compatibility rules
- active instance references

The system must save:
- version snapshots
- change rationale
- compatibility analysis
- migration records
- release notes for process changes

Information changes because:
- process logic evolves
- a stable discovery is promoted
- an evaluator or contract changes meaning

## 8.9 Support hierarchical parent-child processes

The system shall support processes that spawn child processes, including one-to-many relationships.

The system must support:
- parent calling one or many children
- children inheriting selected context from parent
- children producing artifacts back to parent
- parent waiting on, monitoring, or partially proceeding past children
- child retry independent of full parent restart
- parent reactions to child failure based on explicit policy
- visualization of hierarchy in both definition and runtime

The system must read:
- parent instance state
- child invocation contracts
- child completion policies
- child output mappings

The system must save:
- parent-child linkage
- child invocation records
- child result mappings
- child retry lineage
- parent dependency state

Information changes because:
- a child is started, retried, replaced, cancelled, or completed
- the parent route changes based on child outcomes

A parent instance must not be assumed to have only one child instance per child definition. A parent may accumulate multiple child instances over time due to retries, forks, or repeated invocations.

## 8.10 Natural-language-assisted authoring

This is an enhancement, not a foundational requirement.

The system may allow a user to:
- create a draft process from natural language
- update a process using natural language
- explain a process in natural language
- suggest missing contracts, evaluators, or orchestration structures

The system must treat natural-language authoring as a proposal layer that produces explicit structured changes for review, not as a direct uncontrolled editor.

The system must read:
- user instructions
- current process definition
- process library context

The system must save:
- generated draft changes
- structured edit proposals
- acceptance / rejection records

Information changes because:
- the user accepts, rejects, or modifies suggested process edits

---

## 9. Authoring and Visualization Requirements

### 9.1 Graph editing
The system shall provide a graph representation for process definitions, including:
- nodes
- directed edges
- nested subprocesses
- conditional routes
- parallel paths
- convergence points
- visual identification of process type
- visual identification of contract completeness
- visual identification of evaluation gates

### 9.2 Hierarchical navigation
The system shall allow the user to:
- zoom into child processes
- return to parent context
- view parent-child lineage
- understand whether a subprocess is embedded, referenced, or instantiated dynamically

### 9.3 Process diff visualization
The system shall visualize structural changes across versions, including:
- step additions and removals
- route changes
- contract changes
- evaluator changes
- child-process mapping changes

### 9.4 Runtime overlay
The graph view shall support overlaying runtime information such as:
- current active steps
- completed steps
- failed steps
- retries
- branch choices
- evaluator outcomes
- child process status

---

## 10. Discovery and Boundary Testing Requirements

### 10.1 Discovery objects
The system shall model exploratory work explicitly rather than forcing it into formal execution objects too early.

At minimum, the system must support:
- Theory
- Experiment Plan
- Boundary Test
- Variant Process Definition
- Promotion Recommendation

### 10.2 Variant execution
The system shall allow multiple candidate process variants to be executed and compared under controlled conditions.

### 10.3 Boundary measurement
The system shall support recording:
- manipulated variables
- tested values or conditions
- observed effects
- failure thresholds
- soft degradation points
- hard failure points
- confidence notes

### 10.4 Promotion to formal process logic
The system shall allow a discovery result to be promoted into a new or changed formal Process Version only after:
- evidence is recorded
- a rationale is attached
- compatibility impact is checked
- promotion is approved

---

## 11. Runtime and State Model Requirements

### 11.1 Process instance states
At minimum, a Process Instance shall support states such as:
- draft
- queued
- running
- waiting
- paused
- blocked
- retrying
- failed
- cancelled
- completed
- superseded

### 11.2 Step attempt states
At minimum, a Step Attempt shall support:
- pending
- running
- waiting-input
- succeeded
- failed
- retry-scheduled
- skipped
- mocked
- cancelled

### 11.3 State transition recording
Every state transition must create an Audit Event.

### 11.4 Checkpointing
The system shall support checkpoints at process and subgraph levels.

A checkpoint must be restorable subject to policy and must retain enough context to explain:
- what had completed
- what values existed
- what artifacts had been produced
- which branches were active
- which child processes were in scope

---

## 12. Evaluation Requirements

### 12.1 Evaluation as first-class behavior
The system shall support evaluation at:
- step level
- branch level
- child-process level
- whole-process level
- cross-run comparative level

### 12.2 Evaluation outcomes
An evaluation may return at minimum:
- pass
- fail
- inconclusive
- requires-human-review
- blocked-by-missing-evidence

### 12.3 Evaluation evidence
An evaluation result must reference the evidence it used.

### 12.4 Human review integration
A human review is a valid evaluator type and must be recorded with:
- decision
- rationale
- attached evidence
- reviewer identity
- timestamp

---

## 13. Information Read / Save / Change Rules

This section describes what information categories the system must manage.

### 13.1 Information the system must read
The system must be able to read:
- process definitions
- process versions
- step contracts
- evaluator definitions
- runtime input values
- historical execution traces
- artifacts
- child-process outputs
- theory and experiment data
- comparison baselines
- version compatibility rules
- mock configurations
- operator annotations

### 13.2 Information the system must save
The system must save:
- process definitions and versions
- step and process contracts
- process instances
- step attempts
- artifacts
- audit events
- checkpoints
- theory records
- experiment and boundary-test results
- evaluator outcomes
- parent-child relationships
- diff and migration records
- import/export packages
- mock definitions
- annotations and review records

### 13.3 Why information changes
Information changes only for valid reasons. The system shall classify changes under categories such as:
- authoring change
- version change
- runtime transition
- retry or replay
- evaluation outcome
- promotion from discovery to formal process
- human intervention
- external signal
- migration or supersession

Every meaningful change must be attributable to a reason category.

---

## 14. Mocking, Simulation, and Dry-Run Requirements

The system shall support process development without requiring all dependencies to be live.

It must support:
- mocked step outputs
- mocked child-process outputs
- mocked evaluator outcomes
- simulated failures
- dry-run execution without side effects
- replay using historical inputs
- branch exploration from checkpoints

Mock behavior must be explicit and inspectable. A user must never mistake a mock result for a live result.

---

## 15. Import / Export Requirements

The system shall allow portable representation of process definitions and related objects.

Import/export must support at minimum:
- process definitions
- process versions
- contracts
- evaluator definitions
- mock profiles
- selected reference artifacts

Import/export packages must preserve semantic meaning well enough for:
- backup
- transfer
- review
- diff
- reuse

---

## 16. Analytics and Process Intelligence Requirements

The system shall support process intelligence at two levels.

### 16.1 Definition-level intelligence
Used to analyze how a process is designed.

Must include:
- unreachable steps
- missing evaluators
- incomplete contracts
- route complexity indicators
- child-process dependency complexity
- change frequency by area

### 16.2 Runtime intelligence
Used to analyze how a process behaves.

Must include:
- common paths
- rare paths
- failed paths
- retry hotspots
- evaluator rejection hotspots
- high-latency branches
- fragile child-process dependencies
- path stability by version
- common boundary failures

---

## 17. AI Compatibility Requirements

The system shall support AI steps, AI-assisted authoring, and AI-backed evaluators without making AI a special category outside the process model.

### 17.1 AI step parity
AI steps must support the same inspection and lifecycle treatment as any other step.

### 17.2 AI artifact capture
The system should preserve the most relevant AI artifacts needed for audit and replay, such as:
- task instructions
- structured inputs
- context bundle references
- produced outputs
- evaluation results

### 17.3 AI substitution
An AI step should be replaceable with a deterministic step, human step, or alternate AI-capable step if contracts match.

This is required to keep the system architecture stable while AI capability changes over time.

---

## 18. Versioning and Backward Compatibility Requirements

### 18.1 Immutable historical meaning
A historical Process Instance must remain interpretable against the Process Version that created it.

### 18.2 Compatibility analysis
Before activating a new Process Version, the system shall determine whether changes are:
- non-breaking
- conditionally compatible
- breaking

### 18.3 Active instance handling
The system shall define what happens to in-flight instances when a newer Process Version is activated.

Supported behaviors may include:
- remain on original version
- manually migrate
- automatically migrate if compatible
- fork into new instance lineage

### 18.4 Version lifecycle
At minimum the version lifecycle shall support:
- draft
- review
- active
- deprecated
- retired

---

## 19. Parent-Child Execution Semantics

The system must define explicit semantics for parent-child process relationships.

### 19.1 Invocation modes
A parent may invoke a child process as:
- blocking
- non-blocking
- fan-out one-to-many
- conditional
- iterative

### 19.2 Failure handling
Child failure policy must be explicit. Possible outcomes include:
- retry child only
- replace child with alternate child
- escalate to parent review
- fail parent branch only
- fail entire parent instance
- mark parent as waiting for intervention

### 19.3 Data exchange
The system must define:
- what parent input the child receives
- what child outputs are returned
- whether outputs are aggregated, selected, or compared
- whether the parent stores child outputs as artifacts or values or both

### 19.4 Visibility
A parent instance view must allow inspection of:
- all child instances
- child status
- child retry lineage
- child outputs
- child-linked audit events

---

## 20. Minimal Viable Release vs Deferred Features

This section intentionally pushes back on features that are useful but not essential for an effective first release.

## 20.1 Essential for first release
These capabilities should exist early because they unlock real orchestration progress.

- process definitions and immutable versions
- visual graph editor for formal processes
- execution processes with step contracts
- process instances and step attempts
- retries, branching, convergence, pause/resume
- artifacts and audit events
- parent-child process support
- evaluators and human review
- mocking / dry-run execution
- basic discovery objects: Theory, Variant, Boundary Test
- process diff and version comparison
- runtime inspection of all step values

## 20.2 Valuable but can wait
These features are strong enhancements but should not block the first useful system.

- natural-language process creation and editing
- advanced process mining and anomaly clustering
- automated promotion recommendations
- fully generalized boundary-test automation
- rich scorecards for every possible process quality dimension
- highly opinionated global ontology across all domains

The first release should optimize for explicitness, inspectability, and orchestration completeness rather than for maximum intelligence.

---

## 21. Acceptance Criteria for the System as a Whole

The system is considered functionally successful when a user can:
- define a process graph with explicit contracts and evaluators
- run a process instance and inspect every step value and artifact
- retry, replay, or fork execution without losing auditability
- version process definitions safely
- spawn child processes and understand parent-child runtime behavior
- represent an emerging process as theories and variants before formalizing it
- compare exploratory outcomes and promote stable results into formal process logic
- use AI-capable steps without breaking the system model
- continue evolving orchestration without having to build missing foundational capabilities midstream

---

## 22. Summary

The left system is not merely a workflow runner. It is a combined environment for:
- process definition
- process execution
- process discovery
- process comparison
- process evaluation
- process evolution

Its key contribution is that it treats known work and emerging work under one conceptual model while preserving explicitness, auditability, hierarchical composition, and compatibility with changing AI capability.

The system should therefore be built around:
- process definitions and versions
- contracts and evaluators
- instances, attempts, and artifacts
- discovery objects and boundary tests
- parent-child execution semantics
- auditability and replay
- progressive formalization

This is the minimum coherent foundation for a left system that can support real orchestration work without repeatedly forcing the user to stop and build missing orchestration features during the journey.
