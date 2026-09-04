# Agent Orchestration & Process Externalization System

## Business / Product Specification

## Status

This specification has been updated so that the **MVP implementation stages prevail** over earlier product assumptions.

The main consistency corrections are:

- executions may exist before processes
- process graphs are introduced before full execution automation
- process-linked executions come after standalone executions
- human agents come before messenger/inbox
- messenger/inbox controls human execution after human agents exist
- external AI agents are added after human-agent execution is working
- process hierarchy comes after basic human and AI execution
- audit, governance, analytics, versions, and artifacts are foundations or hardening layers, not the first product surface

---

# 1. Product Summary

The product is an **interaction-first, process-centered orchestration system** for turning implicit human and agent workflows into explicit, executable, inspectable processes.

The system starts from the reality that humans already perform implicit processes. The first product step is to let those executions be recorded explicitly. Formal process graphs then become the reusable structure that executions can be spawned from, attached to, inspected against, and improved over time.

The core product experience is built in stages:

1. **xyflow process graph** — create and edit explicit process topology.
2. **Standalone executions** — record work before a process is formalized.
3. **Process workspace** — browse processes, open graphs, and use contextual MCP chat.
4. **Process-execution linking** — spawn executions from processes or attach existing executions to processes.
5. **Human agents** — model humans as responsible actors in the process system.
6. **Messenger/inbox** — let human agents control execution through chat-like interactions.
7. **External AI agents** — let bounded AI agents perform execution steps.
8. **Process hierarchy** — allow parent processes to spawn child executions.
9. **Audit and hardening** — preserve traceability, versions, artifacts, permissions, and performance foundations.

---

# 2. Product Thesis

Most real workflows begin as **implicit executions**.

A person or agent performs work, requests missing information, makes decisions, produces outputs, and moves forward. The process may be real and repeatable, even if no formal process graph exists yet.

The product thesis is:

> Start by making executions explicit, then connect repeated or important executions to formal process graphs, then let humans and agents participate through structured orchestration surfaces.

The system should not force users to fully define a process before recording work. Instead, it should support this progression:

```text
implicit work
→ explicit execution record
→ explicit process graph
→ process-linked executions
→ human-agent control
→ AI-agent execution
→ subprocess hierarchy
→ audit and governance
```

---

# 3. Primary Business Problem

Organizations and individuals increasingly use agents, chats, prompts, scripts, manual handoffs, and ad hoc approvals to get work done. The actual operating process often remains hidden inside:

- people's heads
- chat history
- one-off prompts
- scattered tool calls
- undocumented human decisions
- manual approvals
- invisible retries and exceptions

This creates five business problems:

| Problem | Product Response |
|---|---|
| Work happens before the process is written down | Allow standalone executions before process formalization |
| Processes are implicit | Let executions later become attached to or compared against explicit process graphs |
| Humans are treated as external blockers | Model humans as agents with responsibilities and inbox-driven execution control |
| Agent work is hard to inspect | Store execution steps, inputs, outputs, messages, and actor runs |
| Governance is expensive after the fact | Generate audit data as a side effect of normal orchestration |

---

# 4. Product Positioning

**Category:** Interaction-first process externalization and agent orchestration platform.

**Positioning statement:**

> A system for recording real work executions, formalizing recurring work into explicit process graphs, and orchestrating human and AI agents through inspectable execution records.

The product is not only:

- a workflow automation tool
- an agent runtime
- a process mining tool
- a chat interface
- a task manager

It is a business operating layer that connects:

- implicit work capture
- explicit process definition
- process execution
- human-agent participation
- external AI-agent execution
- future audit and governance

---

# 5. Core Product Principles

## 5.1 Executions can exist before processes

A user must be able to record work that happened or is happening without first creating a formal process.

This matters because the product is meant to externalize implicit processes, not only run already-known processes.

## 5.2 Process graphs become canonical after formalization

Once a process exists, the graph is the canonical orchestration artifact.

MCP, AI, templates, or manual actions may create or modify the graph, but the saved process definition remains structured, inspectable, and executable.

## 5.3 MCP is a control and synthesis layer

MCP can create processes, add nodes, add edges, create executions, update executions, manage agents, and answer questions about process or execution state.

MCP should not replace the structured system of record.

MCP should produce or apply explicit structured changes.

## 5.4 Human agents come before AI agents

The MVP should first support human responsibility and human-controlled execution.

External AI agents should be added only after:

- executions exist
- processes can link to executions
- human agents exist
- messenger/inbox can control human execution

## 5.5 Messenger is an execution-control surface

Messenger is not the first artifact and not the system of record.

Messenger appears after human agents exist. Its job is to let humans respond naturally when a process execution requires human input, review, approval, rejection, clarification, or final response.

## 5.6 AI agents are bounded step actors

External AI agents should perform bounded execution steps.

The process graph owns orchestration. The AI agent owns only the local step output.

## 5.7 Hierarchy prevents one giant process

Complex work should be represented through parent processes and child executions, not by forcing every process into one huge graph.

## 5.8 Audit is a by-product before it is a product

The system should capture events, inputs, outputs, messages, versions, and artifacts early enough to support future audit and analysis.

However, advanced governance dashboards and process analytics should not block the first usable MVP.

---

# 6. MVP Scope by Stage

## Stage 1 — xyflow Graph, Supabase, and MCP Process Creation

### Goal

Create the first explicit process graph surface.

### User Can

- open a process graph page
- create one process through MCP
- create nodes through MCP
- create edges through MCP
- persist the graph in Supabase
- reload the graph with saved positions

### Not Yet Included

- executions
- agents
- messenger
- AI agents
- process hierarchy

---

## Stage 2 — Standalone Executions Without Processes

### Goal

Allow users to explicitly record executions before formal processes exist.

### User Can

- create an execution without a process
- add manual execution steps
- update execution status
- record inputs, outputs, notes, and decisions
- use MCP to create and update executions
- complete an execution without linking it to a process

### Product Rule

Executions must not require `process_definition_id`.

A process-optional execution layer is required.

---

## Stage 3 — Process List and Contextual Process MCP Chat

### Goal

Turn the process graph into a usable process workspace.

### User Can

- browse processes from a left sidebar
- search processes
- click a process to open it
- view the graph in the center
- use a right-side MCP chat scoped to the selected process
- ask MCP to add, edit, connect, rename, or explain process elements

### Not Yet Included

- execution linking
- human agents
- messenger
- AI agents

---

## Stage 4 — Link Processes and Executions

### Goal

Bridge explicit process graphs with concrete executions.

### User Can

- spawn a new execution from a process
- attach an existing standalone execution to a process
- view executions for a process
- search and filter executions for a process
- inspect execution inputs and outputs
- manually move execution steps forward
- ask MCP questions about an execution
- see misalignment between an execution and the process graph

### Product Rule

Advancement remains manual at this stage. No AI agents are connected yet.

### Misalignment Examples

Misalignment should show when:

- execution has extra steps not in the process
- process has missing steps not present in the execution
- execution step order conflicts with process edges
- output shape does not match expected process step output
- a completed execution assigned to a process does not match the process structure

---

## Stage 5 — Human Agents and Responsibility

### Goal

Model humans as first-class agents before adding automation.

### User Can

- open an agents table
- add, edit, delete, or deactivate human agents
- search human agents
- assign responsibility for a process to a human agent
- assign steps to human agents
- use MCP to manipulate human agents and responsibility

### Product Rule

All agents at this stage are humans.

Execution advancement is still manual.

---

## Stage 6 — Messenger / Inbox for Human-Agent Control

### Goal

Let human agents control execution through a consolidated inbox.

### User Can

- open an inbox
- see execution steps requiring their input
- open a message thread
- read the request and process context
- reply, approve, reject, provide data, or ask for clarification
- submit a final response
- update or complete the execution step through the interaction

### Product Rule

A human reply is not merely chat. It should become structured execution output.

---

## Stage 7 — External AI Agent Execution

### Goal

Allow external AI agents to perform bounded execution steps.

### User Can

- register external AI agents
- assign an AI agent to a process step
- run an AI-agent step during execution
- store AI output as step output
- inspect AI run metadata
- route AI output to human review when needed

### Product Rule

AI agents do not own orchestration. They perform local step work inside the explicit execution model.

---

## Stage 8 — Process Hierarchy and Subprocess Executions

### Goal

Allow parent processes to spawn child executions from subprocess steps.

### User Can

- define a process step as a subprocess invocation
- spawn a child execution from a parent execution step
- inspect child execution status from the parent
- navigate parent-child execution lineage
- map child output back to the parent step

### Product Rule

Hierarchy should be represented through linked executions, not by flattening all work into one graph.

---

## Stage 9 — MVP Hardening

### Goal

Add the minimum durable foundations needed to make the MVP reliable.

### Includes

- expanded statuses
- audit events
- process version snapshots
- artifacts
- search and performance optimization
- basic permissions
- import/export
- demo seed data

### Product Rule

These features support trust, traceability, and scale, but advanced governance and analytics remain later product layers.

---

# 7. Core Product Objects

## 7.1 Process Objects

| Object | Meaning |
|---|---|
| Process Definition | Reusable process blueprint represented as a graph |
| Step Definition | Node in the process graph |
| Edge Definition | Directed connection between process steps |
| Process Version | Snapshot of a process at a point in time |
| Subprocess Link | Declaration that a process step may spawn another process |

## 7.2 Execution Objects

| Object | Meaning |
|---|---|
| Execution | One concrete run of work, with or without a linked process |
| Execution Step | Recorded step inside an execution |
| Execution Event | Runtime event, note, status change, or state transition |
| Process-Execution Link | Relationship between a process and an execution |
| Execution Hierarchy Link | Parent-child relationship between executions |

## 7.3 Agent Objects

| Object | Meaning |
|---|---|
| Actor / Agent | Human, AI agent, tool, or system actor |
| Human Agent | Human participant responsible for process or step work |
| AI Agent | External AI-backed actor that performs bounded step work |
| Agent Run | One AI-agent execution attempt |
| Process Responsibility | Assignment of owner, responsible agent, reviewer, or fallback |

## 7.4 Interaction Objects

| Object | Meaning |
|---|---|
| Interaction Thread | Messenger-style exchange linked to an execution or step |
| Message | Structured request, response, note, approval, rejection, or final answer |
| Party | Contextual participant in an interaction |

## 7.5 Evidence and Governance Objects

| Object | Meaning |
|---|---|
| Artifact | Larger or meaningful output produced by an execution or step |
| Audit Event | Append-only record of meaningful changes or runtime events |
| Alignment Report | Comparison between a process graph and an execution |

---

# 8. Main UI Specification

## 8.1 Top-Level Navigation

The product should eventually have these primary areas:

| Area | Introduced | Purpose |
|---|---:|---|
| Process Graph | Stage 1 | Create and edit explicit process topology |
| Executions | Stage 2 | Record and inspect standalone executions |
| Process Workspace | Stage 3 | Browse and edit processes with contextual MCP |
| Process Executions View | Stage 4 | Spawn, attach, inspect, and align executions |
| Agents | Stage 5 | Manage human agents and responsibility |
| Messenger / Inbox | Stage 6 | Let humans control execution steps |
| AI Agent Runs | Stage 7 | Inspect external AI-agent outputs and failures |
| Hierarchy View | Stage 8 | Navigate parent and child executions |
| Audit / Versions / Artifacts | Stage 9 | Support durability and future governance |

---

## 8.2 Process Workspace

Layout after Stage 3:

```text
Left sidebar:
- process search
- process list
- create process

Center:
- xyflow graph
- selected process title
- node/edge selection

Right:
- contextual MCP chat
- selected process context
- selected node/edge context
```

The process graph represents explicit process topology.

It should not render raw message history as graph nodes by default.

---

## 8.3 Execution Workspace

Layout after Stage 2:

```text
Left/list:
- executions
- search/filter
- status

Main:
- execution detail
- manual execution steps
- input/output
- notes/events

Right:
- MCP chat for updating or asking about execution
```

Executions may be processless.

---

## 8.4 Process-Linked Execution View

Layout after Stage 4:

```text
Left:
- executions linked to selected process
- search/filter/pagination

Center:
- selected execution steps
- graph overlay or alignment view

Right:
- MCP execution chat
- input/output questions
- misalignment summary
```

---

## 8.5 Agents Page

Introduced in Stage 5.

Data table should support:

- add human agent
- edit human agent
- delete or deactivate human agent
- search human agents
- assign process responsibility
- assign step responsibility

---

## 8.6 Messenger / Inbox

Introduced in Stage 6.

Inbox rows should show:

- process name, if linked
- execution title
- step title
- requesting party
- assigned human agent
- state
- last message
- unresolved required fields
- updated time

Thread view should show:

- process context
- execution context
- step context
- message timeline
- structured reply controls
- approve / reject / clarify / final response actions

---

# 9. MCP Product Role

MCP is the natural-language control layer for the product.

## 9.1 Stage-Based MCP Scope

| Stage | MCP Capabilities |
|---:|---|
| 1 | Create process, add nodes, add edges, update graph |
| 2 | Create/update executions and execution steps |
| 3 | Contextual process editing and explanation |
| 4 | Spawn/attach executions, map steps, ask execution questions |
| 5 | Create/update agents and assign responsibility |
| 6 | Create/reply to threads and complete steps from messages |
| 7 | Register and run AI agents |
| 8 | Spawn child executions and inspect lineage |
| 9 | Query audit events, versions, artifacts, and status history |

## 9.2 MCP Boundary

MCP should not be the source of truth.

Its role is to:

1. interpret user intent,
2. call structured tools,
3. create or update records,
4. explain state,
5. propose changes,
6. execute approved actions.

The persisted product data remains authoritative.

---

# 10. Business Rules

## 10.1 Execution Rules

1. An execution may exist without a process.
2. An execution may later be attached to one or more process candidates.
3. An execution spawned from a process should generate execution steps from the process steps.
4. Manual execution advancement is the default until human or AI execution control is added.
5. Completed executions can still be attached to processes for comparison and alignment.

## 10.2 Process Rules

1. A process is an explicit graph of steps and edges.
2. A process can be created manually or through MCP.
3. A process is not required before execution recording.
4. A process should become canonical once a workflow is formalized.
5. Structural process changes should eventually create version snapshots.

## 10.3 Process-Execution Alignment Rules

The system should detect and display when:

- execution steps do not map to process steps
- process steps are missing from the execution
- execution contains extra work
- execution order conflicts with process topology
- expected outputs are missing
- completed execution behavior diverges from the assigned process

## 10.4 Agent Rules

1. Human agents are supported first.
2. A process can have responsible human agents.
3. A step can have an assigned actor.
4. AI agents are added only after human-agent workflow exists.
5. AI agents must behave as bounded step actors.

## 10.5 Messenger Rules

1. Messenger threads appear when human input is needed.
2. A thread should be linked to an execution or execution step.
3. A human response can update step output.
4. A final human response can complete a step.
5. Conversation should remain inspectable from the execution.

## 10.6 AI Agent Rules

1. AI agents are external step performers.
2. AI output must be captured as execution output.
3. AI run metadata should be stored.
4. Failed AI runs should be visible.
5. Human review can be inserted after AI output.

## 10.7 Hierarchy Rules

1. A process step may spawn a child execution.
2. Child executions must link back to the parent execution step.
3. Parent execution should show child status.
4. Child output can be mapped back to parent step output.
5. Parent-child lineage must be inspectable.

---

# 11. Use Cases

## 11.1 Record an implicit execution before a process exists

**Scenario:**  
A user handles a customer request manually.

**System behavior:**  
The user creates a standalone execution, records steps, captures inputs and outputs, and marks the execution complete.

**Value:**  
The user externalizes real work without needing to define a process first.

---

## 11.2 Formalize a repeated execution into a process

**Scenario:**  
Several standalone executions follow a similar pattern.

**System behavior:**  
The user creates a process graph and attaches previous executions to compare how well they fit.

**Value:**  
Implicit repeated work becomes explicit reusable process structure.

---

## 11.3 Create a process through MCP

**Scenario:**  
The user says, “Create a customer intake process with intake, missing-info collection, review, and final response.”

**System behavior:**  
MCP creates a process, adds nodes, and connects edges in the xyflow graph.

**Value:**  
Natural language speeds up process authoring while the graph remains structured.

---

## 11.4 Spawn an execution from a process

**Scenario:**  
A process exists and the user wants to run it for a new customer.

**System behavior:**  
The system creates a new execution, links it to the process, and generates execution steps from the process graph.

**Value:**  
The process becomes executable without requiring full automation yet.

---

## 11.5 Attach a completed execution to a process

**Scenario:**  
A user has already completed work manually and later creates a process.

**System behavior:**  
The completed execution is attached to the process and checked for alignment.

**Value:**  
Historical work becomes evidence for process design.

---

## 11.6 Show misalignment between execution and process

**Scenario:**  
A completed execution skipped a step that exists in the process.

**System behavior:**  
The execution view shows missing, extra, or out-of-order steps.

**Value:**  
The system highlights the gap between formal process and real behavior.

---

## 11.7 Assign process responsibility to a human agent

**Scenario:**  
A process needs an accountable owner.

**System behavior:**  
The user creates a human agent and assigns them as the responsible agent for the process.

**Value:**  
Process ownership becomes explicit.

---

## 11.8 Human controls an execution through messenger

**Scenario:**  
A process execution requires approval before continuing.

**System behavior:**  
The assigned human receives an inbox thread, reviews context, approves or rejects, and the response updates the execution step.

**Value:**  
Human-in-the-loop work becomes natural while remaining structured.

---

## 11.9 External AI agent performs a step

**Scenario:**  
An AI coding or research agent should produce an output for one step.

**System behavior:**  
The execution step calls the external AI agent, stores input, output, metadata, and errors, then makes the result inspectable.

**Value:**  
AI becomes a bounded participant in the process rather than an uncontrolled black box.

---

## 11.10 Parent process spawns a child execution

**Scenario:**  
A customer onboarding process needs a separate compliance review subprocess.

**System behavior:**  
A subprocess step spawns a child execution, links it to the parent step, waits for completion, and maps the child output back to the parent.

**Value:**  
Complex work remains modular and inspectable.

---

## 11.11 Audit a completed execution

**Scenario:**  
A manager asks why a decision happened.

**System behavior:**  
The execution view shows steps, messages, inputs, outputs, actor actions, AI runs, and audit events.

**Value:**  
The system can reconstruct what happened without a separate audit effort.

---

# 12. MVP Success Criteria

The MVP is successful when the product can support the following end-to-end flow:

1. A user creates a process graph through MCP and xyflow.
2. A user records a standalone execution before a process exists.
3. A user browses and opens processes from a searchable sidebar.
4. A user spawns an execution from a process.
5. A user attaches an existing execution to a process.
6. The system shows alignment or misalignment between execution and process.
7. A user creates and manages human agents.
8. A user assigns process and step responsibility to human agents.
9. A human agent receives an inbox thread for execution input.
10. A human response updates or completes an execution step.
11. An external AI agent can be registered and called from an execution step.
12. A parent process can spawn a child execution.
13. Important changes and outputs are captured enough for later audit and analysis.

---

# 13. Deferred Product Areas

These remain valuable but should not distort the MVP:

- advanced process mining
- automatic process promotion
- automated governance dashboards
- complex anomaly detection
- full multi-party messaging
- deep semantic clustering
- autonomous multi-agent planning
- full permission model
- complex branching automation
- enterprise analytics
- fully generalized ontology across all process domains

The MVP should prove the core loop first:

```text
record work
→ formalize process
→ link execution to process
→ assign humans
→ control work through messenger
→ add bounded AI agents
→ support subprocess hierarchy
→ preserve evidence
```

---

# 14. Main Product Risk

The biggest product risk is collapsing the system into chat.

Chat should be useful for:

- MCP control
- human replies
- clarifications
- approvals
- explanations

But chat should not replace the structured system of record.

The product should preserve these distinctions:

| Surface | Role |
|---|---|
| Process graph | Formal process structure |
| Execution record | Concrete work instance |
| Execution step | Unit of performed work |
| Messenger thread | Human interaction for a step |
| MCP chat | Natural-language control layer |
| Agent run | Bounded AI or human actor work |
| Audit event | Inspectable system history |

This keeps the product from becoming a generic chat tool and protects the long-term value of auditability, process analysis, and governance.
