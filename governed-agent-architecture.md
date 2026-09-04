# A Governed Agent Architecture

*Forced deliberation, auditable execution, and externalized reasoning for AI-assisted workflows*

---

## What this document is

This is the architectural reference for a hybrid human–LLM agent system built on three layers: **Hermes Agent** as the orchestration runtime, a custom **MCP server** as the trusted tool layer, and **Relay** as the human-in-the-loop interface for both model calls and high-judgment decisions. The system is governed by an explicit planning protocol that forces the LLM to externalize its reasoning before acting.

It is not a faster agent. It is not a more autonomous agent. It is an agent designed to make implicit work explicit, expensive decisions auditable, and routine decisions cheap.

---

## The problem this solves

Most agent architectures optimize for autonomy. The LLM plans, executes, decides, and reports. The human shows up occasionally to clarify or approve. When something goes wrong, the trace is buried in token streams and the reasoning is gone the moment the model produces it.

This is fine when the task is routine. It fails when the task is one you are still figuring out.

Consider the work you actually do day-to-day. You receive a request, develop a plan in your head, execute parts yourself, delegate other parts, hit blockers, replan, finish. The plan was implicit. The reasoning lived in your head. The reasons for delegation choices left no trace. If someone asked you six months later why you handled it that way, you would not remember.

The problem this architecture targets is not *"do the work faster."* It is *"make the work legible, auditable, and learnable from."* Speed and autonomy are deliberately traded away for transparency and structure.

Three concrete pressures motivate this:

1. **Governance.** When work has compliance, accountability, or audit requirements, "the LLM decided" is not an acceptable answer. Every decision needs an owner and a record.

2. **Cost.** Autonomous agents burn tokens. For early-stage work, every token spent on the LLM improvising a plan it could have asked you for is wasted. Human attention is finite but free in marginal terms — the right resource to spend when you have time and not budget.

3. **Learning.** The reasoning behind decisions is the most valuable artifact of complex work, and it is the thing autonomous systems discard most readily. Externalizing decisions creates a corpus you can review, share, and build on.

---

## The architecture

Three layers, each with a distinct role and bounded responsibility.

```
┌────────────────────────────────────────────────────────────────────┐
│                        OPERATOR (you)                              │
│  Planning · Catch-all decisions · Supervising · Auditing           │
└────────────────────────────┬───────────────────────────────────────┘
                             │
                  ┌──────────┴──────────┐
                  │      RELAY UI       │
                  │  4 modes:           │
                  │  PLAN · EXECUTE     │
                  │  DECIDE · AUDIT     │
                  └──────────┬──────────┘
                             │
┌────────────────────────────┴───────────────────────────────────────┐
│                       HERMES AGENT                                 │
│  Conversation loop · Tool dispatch · System prompt enforcement     │
│  Plan adherence wrapper · Cancellation · Progress relay            │
└────────────────────────────┬───────────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
        ┌─────┴──────┐               ┌──────┴───────┐
        │   MODEL    │               │  MCP SERVER  │
        │  PROVIDER  │               │  (your code) │
        │            │               │              │
        │  Copilot   │               │  Tool layer  │
        │  Claude    │               │  Human pool  │
        │  GPT       │               │  Routing     │
        │  (or you   │               │  Scoring     │
        │  via Relay)│               │  Delegation  │
        └────────────┘               └──────┬───────┘
                                            │
                                  ┌─────────┴─────────┐
                                  │  HUMAN WORKERS    │
                                  │  Behind tools as  │
                                  │  implementation   │
                                  └───────────────────┘
```

Each layer has a single coherent responsibility:

- **Hermes** orchestrates conversation turns and tool dispatch. It does not decide policy; it executes a protocol.
- **The MCP server** owns the tool surface. Routing, scoring, delegation, and human worker management all live here. The LLM never sees this complexity.
- **Relay** owns the operator's interactive surface. Planning, supervision, catch-all decisions, and audit views.
- **The model provider** does language model inference. Could be Copilot (recommended for cost), direct Anthropic, OpenAI, or Relay itself with a human acting as the model.
- **The operator** is the human conducting the work, holding policy authority, and being the final source of judgment for anything no tool can resolve.

This separation matters because each layer can be reasoned about, tested, and replaced independently. Swap the model provider without touching the MCP server. Swap the MCP server without touching Relay. Swap Relay's UI without changing the protocol Hermes speaks. Compose differently for different use cases.

---

## Layer 1: Hermes as the orchestration runtime

Hermes is the agent runtime. It receives the operator's request, calls the model, parses the model's response, dispatches tool calls, collects results, and feeds them back into the model on the next turn. Standard agent loop.

What makes Hermes the right choice here:

**Programmatic access.** Hermes exposes itself as a Python library (`from run_agent import AIAgent`) and as an OpenAI-compatible HTTP server. Either way, you can drive it from your own backend without using its TUI.

**MCP support.** Tools live in MCP servers, which Hermes consumes as a first-class capability. Tool descriptions, schemas, and runtime updates flow through the standard MCP protocol — no Hermes-specific tool format.

**Provider flexibility.** Hermes works with Copilot, direct Anthropic, OpenAI, Nous Portal, and any OpenAI-compatible endpoint. Switching providers is one config change. You are not locked to any single model vendor.

**Interactive primitives.** Hermes has built-in support for `clarify` (agent asks the operator a structured question), progress notifications (tools stream status updates), cancellation (Ctrl+C interrupts cleanly), and operator-driven steering mid-task.

**Subagent delegation.** Hermes can spawn subagents via `delegate_task` for parallel sub-workstreams. This is separate from the human-as-tool pattern but composable with it.

What Hermes does **not** do in this architecture:

- It does not plan. Planning is a separate step, owned by the operator (or by Claude consulted by the operator), surfaced through a dedicated tool.
- It does not improvise. The system prompt and tool wrapping force every action through a registered tool. Anything not covered surfaces to the operator.
- It does not own policy. The MCP server enforces what is and is not allowed; Hermes executes the protocol the MCP server defines.

---

## Layer 2: The MCP server as the trusted tool layer

The MCP server is the heart of the system's safety and routing logic. It exposes tools to Hermes; internally, it manages human workers, routing decisions, scoring, delegation chains, and audit logging.

### Tools are capabilities, not workers

A common confusion when designing this layer is to expose one tool per human:

```
ask_alice(question)
ask_bob(question)
ask_carol(question)
```

This is wrong. It forces the LLM to reason about who-does-what, which it is bad at; it leaks information about your team; it requires re-advertising tools when the team changes.

The right shape exposes capabilities:

```
ask_legal_reviewer(question, domain)
ask_code_reviewer(question, language)
ask_research_assistant(question, topic)
```

Each tool's implementation maintains a pool of humans qualified for that capability. When the tool is called, internal logic picks the best-available human (Thompson sampling, weighted random, hand-tuned rules, or a combination) and routes the question. The LLM sees one tool and gets one answer. The human pool can churn freely without re-advertising.

### Routing is deterministic

The routing logic is your code. No LLM is involved in picking which human handles a call. You have full observability over human success rates, response times, recent failures, and current availability. The LLM cannot reason about this data reliably; your code can.

Default to Thompson sampling for capability-pool routing. It naturally explores while exploiting, handles new humans gracefully (they get questions until their score stabilizes), and degrades unreliable humans without ever fully cutting them off (in case their recent failures were noise).

### Progress is a side channel

Long-running tool calls (a human takes five minutes to research a question) need progress updates for two reasons: the operator wants visibility, and the LLM client may time out without keepalive signals.

MCP's `notifications/progress` is the right mechanism. Humans emit status updates from their UI; your server forwards them as progress notifications; Hermes displays them inline and resets timeouts on each notification. The LLM never sees the progress messages — only the final tool result. This separation is correct: progress is for operator UX, not model reasoning.

### Pushback flows up; clarification flows sideways

Two distinct "the human can't just answer" patterns, with different correct handling:

**Pushback** is when the human disagrees with the framing of the question. "This question conflates US and EU contract law" is not a missing-fact problem; it is a question-quality problem the LLM should know about. Surface pushback as a structured tool result with `status: rejected` and a reason. The LLM reasons about it and either reformulates or escalates.

**Clarification** is when the human needs a fact they do not have. "What jurisdiction does this contract apply to?" is a missing-fact problem. Use MCP `elicitation` to ask the operator directly, mid-tool-call. The operator answers, your server passes it to the human, the human now answers, the tool returns. The LLM sees a single tool call that took a bit longer than usual.

Surfacing every clarification through the LLM is expensive and adds noise. Surfacing pushback through the LLM is correct because the model needs to learn from the rejection.

### Delegation between humans

Humans behind tools sometimes recognize that another human is better suited. Three patterns:

1. **Internal delegation** (invisible to LLM). Alice hands the question to Bob inside your MCP server. The tool eventually returns Bob's answer. The LLM never knows Alice was involved.
2. **Surfaced delegation** (LLM sees it). Alice responds with `status: deferred, recommendation: ask_ip_specialist`. The LLM decides what to do.
3. **Operator override**. The operator notices Alice is AFK or struggling and reroutes to Carol via a separate admin interface. Your MCP server resolves Alice's pending tool call and starts a new one for Carol, transparently.

Default to pattern 1 for routine load-balancing. Use pattern 2 for genuine semantic mismatches the model should reason about. Build pattern 3 always — it is your operational override valve.

### Audit logging is non-optional

Every tool call, every routing decision, every delegation, every operator override, every score update, every progress notification — log it all. Without this, you have no governance, just process theater.

Logs need: stable IDs that link tool calls across delegation chains, timestamps, the input payload, the output payload, the human(s) involved, the reasoning for routing decisions, and any operator interventions. Store somewhere queryable (Postgres, SQLite with FTS, whatever you prefer). The audit mode of Relay reads from this store.

---

## Layer 3: Relay as the operator's interface

Relay is the front-end the operator works in. It has four distinct modes, each designed for a specific kind of work:

### PLAN — authoring and validating plans

When the operator submits a request, the first thing that happens is a plan gets produced. The planner is either the operator typing the plan directly, the operator consulting Claude and pasting back a structured plan, or (for templated repeat work) the planner instantiating a saved plan with new parameters.

The PLAN view shows the request, the current world state, the available tools with trust indicators, and a plan editor where steps are constructed. Each step is a tool invocation with concrete arguments and an optional rationale note. Plans are validated before approval — missing arguments, implausible orderings, and references to nonexistent state are surfaced as inline warnings.

A validated plan is an artifact. It has a unique ID, a timestamp, an author, the parameters that produced it, and the steps it contains. It is stored in the audit log before execution begins.

### EXECUTE — supervising live execution with drift detection

Once approved, the plan flows into the EXECUTE view as a node graph. Each step is a node; edges connect steps in execution order; node colors indicate status (pending, in-progress, completed, drifted, failed).

The critical feature is **drift detection**. When Hermes attempts to call a tool that is not the next planned step, or calls a tool with arguments that diverge from the plan, the execution wrapper intercepts it. The drift appears as a dashed-orange node on the canvas. The operator gets three options:

- **Force replan** — the planner is re-invoked with the current state to produce a revised plan
- **Accept this deviation** — the unexpected call is incorporated into the plan and execution continues
- **Block and surface** — the drift is treated as a catch-all event, routing the operator to DECIDE mode

Drift is the central governance feature. Most agent architectures hide drift inside the model's reasoning; this one makes it visually unmissable. If the LLM tries to improvise, you see it.

### DECIDE — answering catch-all decisions

When the system encounters a question no registered tool can handle, it invokes a special tool: `human_decision`. This is the explicit escape hatch from full automation. Every invocation produces a focused, single-decision view in DECIDE mode.

The view shows the context (where in the plan this arose, what came before, why no tool fit) and offers four resolution paths:

1. **Answer directly** — operator types a response
2. **Delegate to a tool** — operator picks a tool from the registry and fills arguments; the tool's result becomes the answer
3. **Consult Claude** — copies the context to clipboard, opens Claude.ai; operator pastes Claude's response back
4. **Recursive plan** — triggers a sub-plan; current execution pauses while the sub-plan runs to completion

Each path is one click. The chosen path and the reasoning are logged. Over time, the operator can review catch-all decisions and identify patterns — recurring questions become candidates for new tools.

### AUDIT — retrospective and learning

The AUDIT view is for non-live work: reviewing past executions, identifying patterns, learning from mistakes.

A list of past plans on the left, a read-only execution graph in the center, an inspector on the right. The operator can scrub through execution chronologically, click into any tool call to see its full payload, see where drifts occurred and how they were resolved, see how long each step took.

Filtering and search support questions like:

- *"Which tools drift most often?"* → tools whose registered descriptions diverge from how the model actually uses them
- *"Which catch-all decisions cluster around similar themes?"* → tool development backlog
- *"Which plans needed replanning?"* → plans whose structure was wrong from the start
- *"What did I decide last time this question came up?"* → personal decision audit

This mode is where the architecture's learning value pays off. Without it, you have a process. With it, you have a process you can improve.

---

## The governance protocol

The protocol that ties the three layers together is enforced at the Hermes system prompt level and reinforced at the MCP server level. Three rules:

**Rule 1: No action without a plan.** Hermes's system prompt requires that the first tool call in any session is `plan`, which returns a structured sequence of subsequent steps. Action tools (`ask_legal_reviewer`, `web_search`, etc.) refuse to execute if no approved plan covers them. The MCP server enforces this independently of the LLM following instructions — even if the model improvises, the MCP server checks against the active plan and rejects unexpected calls.

**Rule 2: No improvisation outside registered tools.** Every action must go through a registered tool. If no tool fits the current need, the model must invoke `human_decision` and surface the question to the operator. The system prompt explicitly forbids the model from "answering as if it knew" anything not provided by a tool result.

**Rule 3: Replanning is acceptable; deviation is not.** When the world surprises the plan, the correct response is to invoke `plan` again with the current state. Deviating from the existing plan without replanning is treated as drift and surfaced to the operator. Replanning is a logged event, not a failure — it is a recognition that the original plan was incomplete.

These rules are enforced through three independent mechanisms:

- **System prompt language** that explicitly forbids the prohibited behaviors
- **Plan adherence wrapper** in your MCP server that validates incoming tool calls against the active plan before executing them
- **Drift detection in Relay** that visually surfaces any plan deviations to the operator

Triple enforcement is deliberate. The model will eventually try to break each individual mechanism; you need overlapping layers.

---

## Cost dynamics

This architecture is designed to be cheap to run at low volumes and to scale costs predictably with usage.

**The model provider** is the major cost. Default to GitHub Copilot if you have a subscription — flat fee, multi-vendor model access (Claude Opus 4.7, GPT-5.5, Gemini), no token-by-token billing. For sensitive work, swap the provider to Relay (you-as-model) for specific turns and pay with your attention instead of money.

**MCP server hosting** is your infrastructure cost. A small VPS or even a local process for solo use.

**Relay** runs locally. No external cost.

**Human workers** (if you have a team behind some tools) are an operational cost, not a software cost. They are doing the work they would otherwise be doing manually — this just structures it.

For a solo operator running this architecture against their own work, the steady-state cost is the Copilot subscription. No per-token billing surprises. No usage-based unpredictability. The deliberate inefficiency of human-in-the-loop replanning is what protects you from runaway autonomous-agent costs.

---

## Strengths and significance

What this architecture gives you that autonomous agents do not:

### Externalized reasoning

Plans are documents. Catch-all decisions are records. Tool routing logic is code you can read. Nothing important happens inside a model's hidden reasoning. The system's behavior is reconstructable from its logs.

For complex work where understanding *why* you did something is as valuable as having done it, this is transformative. You build a corpus of how you approach problems. Other people can review your approaches. Future-you can audit past-you.

### Cost-bounded by design

The architecture cannot run away with token usage because it is structurally constrained to pause for human input at well-defined points: between plan and execution, at every catch-all, at every drift event. There is no path to "the agent ran overnight and racked up $400 in API charges" because the agent cannot run overnight without operator attention.

This is the right tradeoff when your bottleneck is budget or attention rather than time.

### Composable and replaceable layers

Each layer has a clean interface. Swap Hermes for a different agent runtime — the MCP server keeps working. Swap your MCP server for a new one — Relay keeps working. Add or remove human workers — the tool surface stays stable. Switch model providers — nothing else changes.

You are not locked in. The architecture is a set of contracts, not a monolith.

### Drift makes the LLM honest

Most agent failures come from the model improvising outside what it was instructed to do. By making drift visually surfaced and structurally blocked, this architecture turns LLM improvisation from a quiet failure mode into a loud, debuggable event. The model still tries to improvise — it always will — but you see it every time, and you handle it deliberately.

### Trust is built incrementally

You start with everything routing to `human_decision`. Over time, you observe which catch-alls cluster, build tools for them, and shift those decisions to automated tool calls. The trust boundary moves outward as you learn what is safe to delegate. You never have to commit to "I trust this LLM with this whole workflow" upfront.

### Process portability

Once your plans, tools, and decision patterns are externalized, the system itself becomes onboarding material. A new team member can read past plans and decisions to learn how you approach work. They can run the same governance system with their own workers behind it. The architecture transfers; only the human pool needs to change.

---

## Honest tradeoffs and limitations

This architecture is not the right choice for every workload.

### It is slow

A task that an autonomous agent would finish in two minutes might take you twenty in this system. The human-in-the-loop steps — approving plans, answering catch-alls, vetting drift — are the dominant cost. If you need speed above all else, this is the wrong design.

### It requires operator attention

The system cannot run unattended. If you walk away, work stops at the next catch-all or drift event. This is a feature for governance and a bug for "let it run overnight" use cases. Decide which mode you are in before adopting.

### It needs upfront investment

The first few weeks of using this system feel like overhead. Plans take effort to write. Every decision surfaces for review. You are doing more work, not less.

The payoff comes later — once you have a corpus of plans, a library of trusted tools, and a feel for which decisions can be safely templated. The investment is real. Decide whether you are likely to do similar work often enough to amortize it.

### Tool design is the hard part

The architecture's quality is bounded by how well your tools are designed. A tool that is too narrow forces too many catch-alls; a tool that is too broad invites the LLM to use it for things it should not. Getting the tool surface right is iterative work that takes attention.

Plan to spend ongoing effort tuning the tool boundary. Treat tool definitions as living artifacts, not one-time configuration.

### Replanning loops are possible

The model can get into a state where every plan it proposes drifts, every replan produces another bad plan, and the operator becomes a permanent participant in the planning loop. When this happens, the issue is usually that the request is malformed or the available tools cannot accomplish it. The right response is to stop, redesign the request, or extend the tool surface — not to keep replanning.

Build in a replan-count alarm. After three replans on the same request, the system should pause and surface "this is not converging" rather than continuing to loop.

### The model is still the planner

Even when you consult Claude for the plan, the plan's quality depends on the model's ability to break down the problem correctly. For genuinely novel problems where the model has no good prior, you may end up writing plans by hand. That is fine — the system supports it — but it removes the "Claude does the thinking" benefit and leaves you with just the governance overhead.

This architecture is most valuable when the model can produce competent first-draft plans that you then audit. It is least valuable when you cannot trust the model's planning at all and must author everything yourself.

---

## Comparison with alternatives

### vs. raw LLM API usage

A raw LLM call is one shot: prompt in, completion out. No tool use, no memory, no governance. Useful for one-off completions. Not comparable to this architecture, which is built for sustained multi-step work.

### vs. Claude Code (or any autonomous coding agent)

Claude Code is an autonomous agent: you give it a task and it executes. It has tool use, file system access, code execution. It is fast and capable.

Difference: Claude Code optimizes for the developer's time. This architecture optimizes for governance and learning. Claude Code is the right choice when speed matters and the task is well-bounded. This architecture is the right choice when the trace of how the work was done is as valuable as the work itself.

You can use both. Claude Code for the parts you trust to autonomous execution; this architecture for the parts that need supervision.

### vs. running an autonomous Hermes setup

Plain Hermes with model provider, MCP servers, and no governance overlay is faster and cheaper per task. The model plans and executes freely; you intervene only when it asks (`clarify`) or when you Ctrl+C.

Difference: plain Hermes hides reasoning inside model context. The governance overlay externalizes reasoning into plans and catch-all decisions. You pay in attention for the externalization.

Adopt this architecture when the work warrants the audit trail. Use plain Hermes when it does not.

### vs. fully manual work (no LLM)

The baseline. You plan in your head, execute, decide as you go. Free. Highest quality reasoning. Least scalable. Worst learning artifacts.

Difference: this architecture is fully manual work *with externalization scaffolding*. You make the same decisions; you just record them in a way that produces learnable artifacts. The overhead is the writing, not the thinking.

For one-off work that you will not repeat, manual is fine. For work patterns you expect to repeat — and where understanding your own decisions matters — this architecture pays.

---

## Practical setup

A walkthrough of how to actually stand this up, assuming you are starting from scratch.

### 1. Provider configuration

If you have GitHub Copilot:

```bash
pip install hermes-agent
hermes model
# pick GitHub Copilot, run device code flow
# pick a model (claude-opus-4.7 recommended)
```

If you do not, the next-cheapest paths are:

- OpenRouter with prepaid credits (multi-vendor, pay-as-you-go)
- Direct Anthropic API key (note the harness-detection risk if you have a Max subscription you wanted to use instead)

### 2. MCP server skeleton

Build a minimal MCP server in Python with FastMCP. Initial tools:

- `plan(request, context)` — returns a structured plan; initially implemented as "open the planning UI in Relay and wait for the operator to author it"
- `human_decision(question, context, severity)` — surfaces a catch-all to Relay's DECIDE mode and waits for the operator's response
- One or two trusted capability tools to start (e.g. `ask_research_assistant(question, topic)`)

Register the server with Hermes in your config.

### 3. Relay frontend

Build the four-mode frontend (PLAN, EXECUTE, DECIDE, AUDIT) following the design system from the Claude Design starter kit. WebSocket connection to your MCP server for real-time updates; HTTP for plan submission and decision resolution.

### 4. Audit logging

Set up a database (SQLite is fine to start) with tables for: plans, executions, tool calls, decisions, drift events, score updates. Every action through the MCP server writes to it. AUDIT mode reads from it.

### 5. Plan adherence wrapper

Before any tool call in the MCP server executes, check it against the active plan. If it is the next planned step, proceed. If not, log a drift event and refuse the call (forcing the LLM to either replan or escalate). The wrapper goes in your MCP server, not in Hermes — even if the model tries to improvise, the server is the gate.

### 6. Iterative tooling

Run real work through the system. Notice patterns in catch-all decisions. Build new tools for recurring patterns. Tune tool descriptions when the model misuses them. Adjust scoring thresholds when good humans get under-routed.

This is the ongoing work. The architecture is not a one-time setup; it is a system you maintain and refine. Plan for that.

---

## Closing notes

This architecture is opinionated. It assumes governance and auditability matter more than autonomy and speed. It accepts the operator as a permanent participant in the loop. It pays in attention for the structure it produces.

For the right problem — work where understanding *why* something was done is as valuable as it being done, where governance is a real requirement, where you want to externalize your own implicit processes into something you and others can learn from — it is a genuinely useful design.

For the wrong problem — high-throughput, low-stakes, well-understood work where speed dominates — it is overhead with no payoff.

Decide which kind of work you are doing before you decide whether to adopt it.

---

## Reference index

- Relay starter kit (Claude Design import): the original four-mode UI design
- Governance revision prompt: the update that added PLAN / EXECUTE / DECIDE / AUDIT and drift detection
- Hermes documentation: https://hermes-agent.nousresearch.com/docs/
- MCP specification: https://modelcontextprotocol.io/
- Model Context Protocol elicitation reference for clarification flows
- MCP progress notifications spec for streaming tool status

---

*This document describes the architecture as designed. Implementation choices, tradeoffs, and edge cases will shift as you actually build it. Update this document as the system evolves — it is a living reference, not a fixed spec.*
