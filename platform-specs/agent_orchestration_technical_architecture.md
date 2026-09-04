# Agent Orchestration & Process Externalization — MVP Technical Implementation

**Date:** 2026-04-24  
**Target stack:** React, MUI, xyflow, Supabase/Postgres, Supabase Edge Functions, Postgres functions, Node worker, MCP, Claude Desktop / Claude Code

---

# Section 1 — Overarching Technical Implementation

## 1.1 Purpose

This document describes the MVP implementation for a personal-use Agent Orchestration & Process Externalization system.

This is not the final enterprise architecture. The goal is to build something useful quickly, keep the physical project count small, and allow the structure to be refined later as the product grows.

The MVP should let me:

1. create process graphs;
2. record executions before a process exists;
3. link executions to processes;
4. assign steps to agents;
5. let agents produce messages, outputs, and execution results;
6. inspect what happened through execution steps, messages, events, and agent runs;
7. use MCP heavily for creating, editing, querying, and running the system.

The central rule is:

> The process does not care whether the assigned agent is human, AI-backed, tool-backed, or system-backed. It only cares that the assigned task is executed and a value/output is returned.

---

## 1.2 MVP Architecture Philosophy

This MVP should optimize for speed, clarity, and personal iteration.

Do not over-design the project structure.

Do not create internal shared packages.

Do not add auth, full audit, enterprise permissions, strict validation layers, or highly separated domain services yet.

Start with:

```text
frontend/       React + MUI UI
supabase/       database, migrations, SQL functions, Edge Functions as backend API
worker/         long-running automated agent tasks
mcp-server/     Claude/MCP tools over the system
docs/           product and technical notes
scripts/        setup, seed, smoke tests
```

The database is the main contract between projects. If similar types or helpers are needed in multiple projects, copy them for now.

---

## 1.3 System-of-Record Rule

Supabase/Postgres is the source of truth.

Everything else is an interface or runtime around it.

| Layer | MVP role |
|---|---|
| React frontend | Main UI for graphs, executions, agents, inbox, and inspection |
| Supabase Edge Functions | Lightweight backend/API for CRUD and orchestration actions |
| Postgres functions | Aggregated reads and database-local queries |
| Worker | Long-running automated agent work |
| MCP server | Natural-language tool interface for Claude Desktop / Claude Code |

For this MVP, a separate Express backend is not necessary unless Supabase Edge Functions become too limiting.

Because the worker handles long-running tasks, Supabase Edge Functions can act as the backend for normal short actions:

- create process;
- add/edit graph nodes;
- create execution;
- update execution step;
- create agent;
- create interaction thread;
- enqueue automated agent job;
- inspect status;
- call Postgres RPCs.

---

## 1.4 Backend Choice: Supabase Edge Functions Instead of Express

For this MVP, use Supabase Edge Functions as the backend layer.

This makes sense because:

1. the system already depends on Supabase/Postgres;
2. most backend actions are short CRUD/orchestration actions;
3. long-running tasks are handled by the worker;
4. there is no auth requirement yet;
5. there is no need for a large Express API surface at this stage.

Recommended approach:

```text
frontend
→ Supabase Edge Function API
→ Postgres tables / RPCs

mcp-server
→ Supabase Edge Function API
→ Postgres tables / RPCs

worker
→ polls jobs from Postgres
→ writes agent runs/messages/outputs back to Postgres
```

Use an Express backend later only if the API grows enough that Deno Edge Functions become uncomfortable.

---

## 1.5 Validation, Auth, and Audit Scope

### Auth

Do not implement auth in the MVP.

Assume single-user / trusted local use for now.

Add auth later when needed.

### Validation

Do not build a heavy validation layer yet.

Use minimal checks where they make the UI or functions easier to debug, for example:

- required IDs are present;
- a process exists before adding a step;
- an execution exists before adding an execution step;
- a job type is known before enqueueing it.

Let Postgres constraints reject invalid data where possible.

The database should carry the most important integrity rules:

- foreign keys;
- not-null constraints;
- unique constraints;
- basic default values.

### Audit

Do not implement a separate `audit_events` table for now.

For the MVP, inspection comes from normal product records:

- `execution_events`;
- `execution_steps`;
- `interaction_threads`;
- `interaction_messages`;
- `agent_runs`;
- `artifacts` if needed later.

Add a dedicated audit system later if this becomes important.

---

## 1.6 Unified Agent Model

There is only one product concept: **agent**.

An agent can be:

- human-driven;
- AI-backed;
- tool-backed;
- system-backed.

But from the process perspective, they are the same:

```text
process step
→ assigned agent
→ task/request is created
→ agent eventually returns output
→ execution step records the result
```

The only difference is how the agent is driven.

| Agent driver | How work happens |
|---|---|
| Human-driven | A human opens the inbox/thread, reads context, replies, approves, rejects, or submits output manually |
| Automated | A worker reads the assigned task, calls a model/tool/provider, writes messages/output automatically |

Both types can have:

- assigned execution steps;
- inbox-style threads;
- messages;
- structured outputs;
- execution events;
- run history;
- artifacts;
- inspection/audit views.

Do not create separate human-agent and AI-agent product flows.

Use one flow:

```text
Execution step needs agent work
→ create interaction thread for assigned agent
→ if agent is human-driven, wait for manual response
→ if agent is automated, enqueue worker job
→ agent response is written as messages and/or output
→ execution step output is updated
→ execution step can be completed
```

---

## 1.7 Runtime Flows

### Process authoring flow

```text
User or MCP
→ Supabase Edge Function
→ insert/update process_definitions, step_definitions, edge_definitions
→ frontend reloads graph through RPC/function
```

### Execution capture flow

```text
User or MCP
→ Supabase Edge Function
→ create execution
→ add execution_steps and execution_events
→ execution can remain processless
```

### Process-execution linking flow

```text
User or MCP
→ Supabase Edge Function
→ create process_execution_link
→ optionally map execution_steps to step_definitions
→ compute simple alignment summary
```

### Unified agent task flow

```text
Execution step assigned to an agent
→ create or reuse interaction_thread
→ add request message with task context
→ if agent has automated connector, enqueue run_agent_step job
→ if agent is human-driven, show thread in inbox
→ agent produces messages/output
→ output updates execution_steps.output_payload
→ step status becomes completed, failed, blocked, or waiting
```

### Worker flow for automated agents

```text
Worker polls orchestration_jobs
→ locks run_agent_step job
→ loads execution step, assigned agent, thread, and connector
→ calls provider/tool/model
→ writes agent_runs record
→ writes interaction_messages record
→ updates execution_steps.output_payload
→ marks job completed or failed
```

### MCP flow

```text
Claude / MCP client
→ MCP tool matched
→ MCP server calls Supabase Edge Function or Postgres RPC
→ function/RPC reads or writes Supabase
→ result returned to Claude
```

If a tool needs long-running work, MCP should not call the worker directly. It should call an Edge Function that creates a job. The worker then picks up the job.

---

## 1.8 Core Data Model

Keep the schema practical and MVP-oriented.

Do not include dedicated audit tables, complex permissions, or process version snapshots yet.

---

## Process Tables

### `process_definitions`

```sql
create table process_definitions (
  id uuid primary key default gen_random_uuid(),
  key text unique not null,
  name text not null,
  description text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

### `step_definitions`

```sql
create table step_definitions (
  id uuid primary key default gen_random_uuid(),
  process_definition_id uuid not null references process_definitions(id) on delete cascade,
  assigned_agent_id uuid references agents(id),
  step_key text not null,
  name text not null,
  description text,
  step_type text not null default 'standard',
  expected_input_schema jsonb,
  expected_output_schema jsonb,
  x_position numeric not null default 0,
  y_position numeric not null default 0,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  unique(process_definition_id, step_key)
);
```

### `edge_definitions`

```sql
create table edge_definitions (
  id uuid primary key default gen_random_uuid(),
  process_definition_id uuid not null references process_definitions(id) on delete cascade,
  source_step_id uuid not null references step_definitions(id) on delete cascade,
  target_step_id uuid not null references step_definitions(id) on delete cascade,
  routing_type text not null default 'default',
  condition_expression text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  unique(source_step_id, target_step_id, routing_type)
);
```

---

## Execution Tables

### `executions`

Executions can exist without a process.

```sql
create table executions (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  summary text,
  status text not null default 'in_progress',
  origin_type text not null default 'manual',
  initial_input jsonb not null default '{}',
  final_output jsonb,
  started_at timestamptz,
  completed_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

### `process_execution_links`

Use a link table instead of requiring `process_definition_id` on `executions`.

```sql
create table process_execution_links (
  id uuid primary key default gen_random_uuid(),
  process_definition_id uuid not null references process_definitions(id) on delete cascade,
  execution_id uuid not null references executions(id) on delete cascade,
  link_type text not null default 'attached',
  alignment_status text not null default 'unmapped',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  unique(process_definition_id, execution_id)
);
```

### `execution_steps`

```sql
create table execution_steps (
  id uuid primary key default gen_random_uuid(),
  execution_id uuid not null references executions(id) on delete cascade,
  step_definition_id uuid references step_definitions(id),
  assigned_agent_id uuid references agents(id),
  title text not null,
  description text,
  status text not null default 'assigned',
  sort_order int not null default 0,
  input_payload jsonb not null default '{}',
  output_payload jsonb,
  notes text,
  alignment_status text default 'unmapped',
  alignment_notes text,
  started_at timestamptz,
  completed_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

### `execution_events`

Use this as the lightweight MVP history/event log.

```sql
create table execution_events (
  id uuid primary key default gen_random_uuid(),
  execution_id uuid not null references executions(id) on delete cascade,
  execution_step_id uuid references execution_steps(id) on delete set null,
  agent_id uuid references agents(id),
  event_type text not null,
  payload jsonb not null default '{}',
  created_at timestamptz not null default now()
);
```

---

## Agent Tables

### `agents`

Use `agents`, not separate human/AI concepts.

```sql
create table agents (
  id uuid primary key default gen_random_uuid(),
  key text unique not null,
  name text not null,
  description text,
  driver_type text not null default 'human',
  active boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

Suggested `driver_type` values:

```text
human
automated
system
tool
```

The process should not branch into different logic based on agent type except for how work gets triggered.

### `agent_profiles`

Optional details for any agent.

```sql
create table agent_profiles (
  id uuid primary key default gen_random_uuid(),
  agent_id uuid not null references agents(id) on delete cascade,
  email text,
  display_name text,
  role_title text,
  timezone text,
  metadata jsonb not null default '{}',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

### `agent_connectors`

Only needed for automated/tool/system agents.

```sql
create table agent_connectors (
  id uuid primary key default gen_random_uuid(),
  agent_id uuid not null references agents(id) on delete cascade,
  provider text not null,
  model_or_agent_ref text,
  configuration jsonb not null default '{}',
  is_active boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

### `agent_runs`

Stores automated run attempts.

A human-driven agent usually will not create `agent_runs`; their work is represented by interaction messages and execution events.

```sql
create table agent_runs (
  id uuid primary key default gen_random_uuid(),
  execution_id uuid not null references executions(id) on delete cascade,
  execution_step_id uuid not null references execution_steps(id) on delete cascade,
  interaction_thread_id uuid references interaction_threads(id) on delete set null,
  agent_id uuid not null references agents(id),
  provider text,
  input_payload jsonb not null default '{}',
  output_payload jsonb,
  status text not null default 'queued',
  error_payload jsonb,
  started_at timestamptz,
  completed_at timestamptz,
  created_at timestamptz not null default now()
);
```

---

## Interaction Tables

### `interaction_threads`

Threads are agent inbox items.

They are not human-only.

```sql
create table interaction_threads (
  id uuid primary key default gen_random_uuid(),
  execution_id uuid not null references executions(id) on delete cascade,
  execution_step_id uuid references execution_steps(id) on delete set null,
  assigned_agent_id uuid references agents(id),
  is_completed boolean not null default false,
  summary text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

### `interaction_messages`

Messages represent requests, replies, observations, approvals, rejections, final outputs, or automated agent responses.

```sql
create table interaction_messages (
  id uuid primary key default gen_random_uuid(),
  interaction_thread_id uuid not null references interaction_threads(id) on delete cascade,
  sender_agent_id uuid references agents(id),
  message_type text not null,
  payload jsonb not null default '{}',
  content text,
  expected_shape jsonb,
  sequence_number int not null,
  created_at timestamptz not null default now()
);
```

Suggested `message_type` values:

```text
request
reply
final_response
approval
rejection
clarification_request
observation
automated_result
error
```

---

## Worker Job Table

```sql
create table orchestration_jobs (
  id uuid primary key default gen_random_uuid(),
  job_type text not null,
  status text not null default 'queued',
  payload jsonb not null default '{}',
  attempts int not null default 0,
  max_attempts int not null default 3,
  run_after timestamptz not null default now(),
  locked_at timestamptz,
  locked_by text,
  error_payload jsonb,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

MVP job types:

```text
run_agent_step
retry_agent_step
generate_alignment_report
spawn_child_execution
map_child_output_to_parent_step
```

---

## Optional Later Tables

Defer these until needed:

- `audit_events`;
- `process_versions`;
- `artifacts`;
- `identity_actor_links`;
- full permission tables;
- advanced analytics tables.

---

## 1.9 Status Model

Keep statuses intentionally minimal for the MVP.

### Execution statuses

```text
in_progress
completed
```

### Execution step statuses

```text
assigned
in_progress
completed
```

### Interaction thread completion

Do not use a thread status enum for now.

Use a boolean flag:

```text
is_completed boolean
```

### Agent run statuses

```text
queued
running
complete
cancelled
```

---

## 1.10 Postgres Functions / RPC

Use Postgres functions for read-heavy or aggregated queries.

Recommended MVP functions:

```sql
get_process_graph(process_id uuid)
get_execution_detail(execution_id uuid)
get_process_execution_summary(process_id uuid)
get_agent_inbox(agent_id uuid)
get_thread_detail(thread_id uuid)
get_execution_lineage(execution_id uuid)
search_processes(search_query text)
search_executions(search_query text)
```

Keep complex business orchestration in Edge Functions or the worker, not in SQL.

---

## 1.11 MCP Implementation

MCP is one of the most important parts of this product.

The MCP server should expose tools that let Claude create, edit, inspect, and execute the system.

### Should MCP call the backend and worker?

For the MVP:

- MCP tools should call Supabase Edge Functions for state-changing actions.
- MCP tools can call Postgres RPCs for read-heavy context.
- MCP should not call the worker directly in normal use.
- If MCP triggers long-running work, it should call an Edge Function that inserts an `orchestration_jobs` row.
- The worker should pick up that job from the database.

So the usual path is:

```text
MCP tool
→ Supabase Edge Function or Postgres RPC
→ database write/read
→ optional job row
→ worker picks up job if needed
```

### MCP tools

Use unified agent naming.

```text
create_process
list_processes
get_process_graph
update_process
create_step
update_step
move_step
delete_step
create_edge
delete_edge

create_execution
list_executions
get_execution
update_execution
add_execution_step
update_execution_step
complete_execution_step
complete_execution

attach_execution_to_process
spawn_execution_from_process
map_execution_step_to_process_step
get_alignment_summary

create_agent
list_agents
get_agent
update_agent
deactivate_agent
configure_agent_connector
assign_agent_to_step

create_agent_thread
list_agent_inbox
get_thread
reply_to_thread
submit_final_response
complete_step_from_thread

run_agent_step
get_agent_run
retry_agent_run

spawn_child_execution
get_execution_lineage
map_child_output_to_parent_step
```

### MCP resources

```text
process://{process_id}/graph
execution://{execution_id}/detail
thread://{thread_id}
agent://{agent_id}
agent://{agent_id}/inbox
```

---

## 1.12 MVP Definition of Done

The MVP is useful when it can do this loop:

```text
Create process graph
→ Record standalone execution
→ Link execution to process
→ Assign any step to an agent
→ Create an agent thread
→ Human-driven agent can respond manually
→ Automated agent can respond through worker
→ Execution step receives structured output
→ User can inspect steps, messages, events, and agent runs
→ MCP can create, update, query, and trigger the above
```

---

# Section 2 — Physical Projects Included in the System

The physical projects should stay simple.

Recommended repository shape:

```text
agent-orchestration/
  frontend/
  supabase/
  worker/
  mcp-server/
  docs/
  scripts/
```

No internal shared packages.

If the same type or helper is needed in multiple projects, copy it.

---

## 2.1 `frontend/` — React + MUI Application

### Purpose

The frontend is the main UI for using and inspecting the system.

### Stack

- React
- TypeScript
- MUI
- xyflow / React Flow
- Supabase browser client, where useful
- simple local API client for Edge Function calls

### Folder structure

Keep this simple. Do not use a feature-based split yet.

```text
frontend/
  src/
    main.tsx
    App.tsx
    routes.tsx
    api.ts
    types.ts
    constants.ts
    theme.ts
    screens/
      ProcessWorkspaceScreen.tsx
      ExecutionListScreen.tsx
      ExecutionDetailScreen.tsx
      AgentsScreen.tsx
      InboxScreen.tsx
      ThreadDetailScreen.tsx
      AgentRunsScreen.tsx
      HierarchyScreen.tsx
    components/
      AppShell.tsx
      ProcessSidebar.tsx
      ProcessGraph.tsx
      StepInspector.tsx
      ExecutionStepList.tsx
      AgentSelector.tsx
      AgentTable.tsx
      InboxList.tsx
      ThreadView.tsx
      MessageComposer.tsx
      JsonEditor.tsx
      StatusChip.tsx
    lib/
      supabase.ts
      format.ts
      graph.ts
```

### Frontend responsibilities

- Render the process graph with xyflow.
- Use MUI for layout, tables, drawers, buttons, inputs, dialogs, and status chips.
- Show process list/search.
- Show execution list/detail.
- Show step inputs and outputs.
- Show agent list and connector config.
- Show inbox threads for agents.
- Show automated agent runs.
- Call Supabase Edge Functions through `api.ts`.
- Call MCP chat surface if added later.

### Frontend non-goals for MVP

- No complex state framework unless needed.
- No feature-folder architecture yet.
- No strict frontend validation layer.
- No auth screens yet.
- No enterprise permissions UI.

---

## 2.2 `supabase/` — Database, RPC, and Edge Function Backend

### Purpose

Supabase is the database and the lightweight backend.

There is no separate Express backend for now.

### Folder structure

```text
supabase/
  migrations/
    0001_agents.sql
    0002_process_graph.sql
    0003_executions.sql
    0004_interactions.sql
    0005_agent_runs_and_jobs.sql
    0006_hierarchy.sql
    0007_indexes.sql
    0008_rpc_functions.sql
  functions/
    api/
      index.ts
      processes.ts
      executions.ts
      agents.ts
      interactions.ts
      agentRuns.ts
      hierarchy.ts
      jobs.ts
  seed.sql
  config.toml
```

### Edge Function API style

Use one `api` Edge Function at first, with simple internal routing.

Example paths:

```text
/functions/v1/api/processes
/functions/v1/api/processes/:id/graph
/functions/v1/api/executions
/functions/v1/api/executions/:id/steps
/functions/v1/api/agents
/functions/v1/api/threads
/functions/v1/api/agent-runs
/functions/v1/api/jobs
```

This is easier than creating many tiny functions early.

Split later if the file grows too large.

### Supabase responsibilities

- Own all tables.
- Own database constraints.
- Own query RPCs.
- Handle short API actions through Edge Functions.
- Insert worker jobs.
- Return data to frontend and MCP server.

### Supabase non-goals for MVP

- No RLS/auth yet.
- No full audit system yet.
- No complex permissions yet.
- No long-running AI/model calls inside Edge Functions.

---

## 2.3 `worker/` — Long-Running Agent Worker

### Purpose

The worker handles long-running automated agent work.

It should not be used for human-driven agents. Human-driven agents interact through the inbox and UI.

### Stack

- Node.js
- TypeScript
- Supabase service client
- Provider SDKs as needed
- simple polling loop over `orchestration_jobs`

### Folder structure

```text
worker/
  src/
    index.ts
    config.ts
    supabase.ts
    jobs.ts
    types.ts
    handlers/
      runAgentStep.ts
      retryAgentStep.ts
      generateAlignmentReport.ts
      spawnChildExecution.ts
      mapChildOutputToParentStep.ts
    providers/
      anthropic.ts
      openai.ts
      genericHttpTool.ts
    utils/
      backoff.ts
      logger.ts
```

### Worker responsibilities

- Poll queued jobs.
- Lock jobs.
- Run automated agent tasks.
- Write `agent_runs`.
- Write `interaction_messages` for automated responses/errors.
- Update execution step output.
- Mark jobs completed or failed.

### Worker non-goals for MVP

- No public API required.
- No Express server required unless you want a health endpoint.
- No separate worker dashboard.
- No complex queue system unless Postgres polling becomes limiting.

---

## 2.4 `mcp-server/` — MCP Tool Server

### Purpose

The MCP server lets Claude Desktop / Claude Code operate the product through tools.

It should be a separate project because MCP has its own runtime and client configuration.

### Does MCP call backend and worker?

MCP should call the Supabase Edge Function API for mutations and Postgres RPCs for reads.

MCP should not call the worker directly in the normal path.

For long-running work:

```text
MCP run_agent_step tool
→ calls Supabase Edge Function
→ Edge Function inserts orchestration_jobs row
→ worker picks up job
→ MCP can later query job/run/thread status
```

### Folder structure

```text
mcp-server/
  src/
    index.ts
    server.ts
    config.ts
    apiClient.ts
    types.ts
    tools/
      processTools.ts
      executionTools.ts
      agentTools.ts
      interactionTools.ts
      runTools.ts
      hierarchyTools.ts
    resources/
      processGraphResource.ts
      executionResource.ts
      threadResource.ts
      agentResource.ts
    prompts/
      processAuthoring.ts
      executionCapture.ts
      alignmentReview.ts
      agentTaskReview.ts
```

### MCP responsibilities

- Expose tools for process graph creation/editing.
- Expose tools for execution capture.
- Expose tools for unified agent creation and assignment.
- Expose tools for inbox/thread actions.
- Expose tools to trigger automated agent runs.
- Expose resources for graph, execution, agent, and thread context.

### MCP non-goals for MVP

- Do not write directly to the database unless it is a local-dev shortcut.
- Do not run long-running model/tool calls itself.
- Do not implement a separate product permission model.

---

## 2.5 `docs/`

```text
docs/
  product_spec.md
  mvp_stages.md
  technical_implementation.md
  schema_notes.md
  mcp_notes.md
```

Keep docs lightweight and update as implementation changes.

---

## 2.6 `scripts/`

```text
scripts/
  reset-db.sh
  seed-demo-data.ts
  smoke-create-process.ts
  smoke-create-execution.ts
  smoke-run-agent-step.ts
```

Scripts are for setup, seed data, and quick manual validation.

---

## 2.7 Physical Project Dependency Rules

Allowed:

```text
frontend → Supabase Edge Function API
frontend → Supabase anon client if useful
mcp-server → Supabase Edge Function API
mcp-server → Postgres RPC/read endpoints if useful
worker → Supabase service client
Supabase Edge Functions → Postgres tables/RPCs
```

Avoid for MVP:

```text
frontend → worker
frontend → direct long-running AI calls
mcp-server → worker direct calls
worker → frontend
worker → mcp-server
any project → shared internal package
```

The worker should communicate through the database/job table, not through frontend or MCP.

---

## 2.8 MVP Build Order

Build the MVP in a sequence that keeps the product usable at every stage. The goal is not to create a perfect architecture first; the goal is to make each layer useful enough that I can start operating the system, observe what feels wrong, and then refine the structure.

### Stage 1 — Database and graph UI

Start by creating the database tables and visual graph surface. This stage proves that a process can exist as structured data and be rendered as an editable graph.

```text
supabase/
  agents table
  process_definitions
  step_definitions
  edge_definitions
  get_process_graph RPC
  api function for process/graph actions

frontend/
  MUI app shell
  process graph screen
  process sidebar
  step inspector

mcp-server/
  create_process
  create_step
  create_edge
  get_process_graph
```

At the end of this stage, I should be able to create a process through MCP or the UI, add steps and edges, save positions, and reload the graph.

### Stage 2 — Executions

Next, add executions without requiring a process. This preserves the product idea that real work can be recorded before a formal process exists.

```text
supabase/
  executions
  execution_steps
  execution_events
  process_execution_links
  execution APIs

frontend/
  execution list screen
  execution detail screen
  manual execution step editor

mcp-server/
  create_execution
  add_execution_step
  update_execution_step
  attach_execution_to_process
```

At the end of this stage, I should be able to record work manually, add steps, mark steps completed, and optionally attach the execution to a process.

### Stage 3 — Unified agents and inbox

Then add agents and the shared inbox/thread model. This is where human-driven and automated agents begin to look the same from the process perspective.

```text
supabase/
  agents
  agent_profiles
  agent_connectors
  interaction_threads
  interaction_messages

frontend/
  agents screen
  inbox screen
  thread detail screen

mcp-server/
  create_agent
  assign_agent_to_step
  create_agent_thread
  reply_to_thread
  submit_final_response
```

At the end of this stage, I should be able to assign a step to an agent, create a thread for that agent, inspect messages, and submit a final response that updates the execution step output.

### Stage 4 — Automated agent runs

After the shared agent/inbox model works, add automation. Automated agents should not be a separate product path; they should consume the same assigned steps and interaction threads that human-driven agents use.

```text
supabase/
  agent_runs
  orchestration_jobs
  job API

worker/
  job polling
  run_agent_step handler
  provider integrations

frontend/
  agent runs screen
  inspect automated messages and outputs

mcp-server/
  run_agent_step
  get_agent_run
  retry_agent_step
```

At the end of this stage, I should be able to configure an automated agent, enqueue a run, let the worker process it, and inspect the resulting messages, run record, and execution step output.

### Stage 5 — Hierarchy and refinement

Finally, add simple hierarchy support so one execution can spawn another. Keep this minimal until the execution and agent loops feel right.

```text
supabase/
  process_subprocess_links
  execution_hierarchy_links

frontend/
  hierarchy screen

worker/
  optional child execution jobs

mcp-server/
  spawn_child_execution
  get_execution_lineage
```

At the end of this stage, I should be able to spawn a child execution from a parent step, inspect the relationship, and map child output back to the parent.

---

## 2.9 Final MVP Rule

Keep the system simple enough to change.

The MVP architecture is:

```text
frontend = MUI UI
supabase = database + backend functions + SQL queries
worker = long-running automated agent jobs
mcp-server = Claude/MCP interface
docs = notes
scripts = setup and smoke tests
```

Do not build enterprise architecture early.

Do not split human agents and AI agents into separate product models.

Do not add auth, audit, permissions, or heavy validation until the product behavior is working.

