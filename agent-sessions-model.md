# Agent Sessions as a Behavioral Model

*A representation in which interactions between agents — not source code — are the primary artifact. Code, tech specs, and product specs are all compilation targets produced from the same session graph.*

---

## 1. Thesis

Software is normally written as source code, and its behavior is an emergent, implicit consequence of that code. This model inverts the relationship. **Behavior is described directly, as a graph of interactions between agents, and source code becomes one of several things a transformer can emit from it.**

The guiding analogy: we stopped hand-writing assembly because a higher-level language expressed human intent more directly and let a compiler handle the mechanical residue. Here, *sessions* are the higher-level language and *code* is the assembly. The long-term goal is to interact with the session graph regularly and treat emitted code as a build output rather than the thing you read and maintain.

The graph is not a one-to-one mapping to any particular codebase, and that is intentional. As long as the interactions are honored, different transformers may emit different code at different levels of conciseness.

---

## 2. Primitives

**Agent.** A component with identity and encapsulated state, reachable only through its operations. Agents can represent anything at any granularity: a `View`, a `ViewModel`, an `APIService`, a single parameter/property, or a unit of control-flow logic.

**Operation (op).** A named entry point on an agent. An op declares the input fields it expects from the caller via a typed `params_schema` (e.g. `{ type: "object", properties: { prompt: { type: "string" } }, required: ["prompt"] }`).

**Session.** An interaction between agents — the visible message traffic. A caller opens a session with a target agent and sends an op; the target eventually returns a response. The session records only what crosses the boundary.

**Dispatch.** Work an agent performs *inside* a session in response to an op. Dispatches can themselves be sessions with other agents, forming a nested tree. A dispatch is not visible to the original caller; it is the agent's private implementation. The full tree of sessions-plus-dispatches resembles the internal workings of functions.

**Variable.** A placeholder (e.g. `{{message}}`) that threads runtime values through the graph and binds dynamic values where an agent genuinely needs them to function. Values are recorded only at agent calls and responses — never for purely local steps.

**Agent-as-argument.** An agent may be passed into a session as a parameter, extending `params_schema` from data to agents. This is dependency injection: the receiver is coupled only to the contract, so any conforming agent — including a mock — can be substituted.

---

## 3. Core principles

**Only observable interactions are modeled.** A line of code that neither reads nor writes outside its own scope carries no interaction meaning and is deliberately absent. From an interaction standpoint it does not matter whether code instantiates a value on one line and passes it on another; only agent calls and responses record values. Characters a language needs to make that work are a code limitation and should not surface in the model.

**Sessions are the source of truth.** Code is a target, not the canonical form. Two session graphs that produce the same interactions describe the same system.

**Granularity is a choice, not a constraint.** The author decides how finely to decompose — from coarse service-level agents down to one agent per variable — applying detail only where it earns its place.

---

## 4. The visibility model (three distinct scopes)

These operate at different levels and must not be conflated:

1. **In-model, agent-to-agent.** A `View` that calls a `ViewModel` op sees only the response, never the ViewModel's internal dispatches. This is encapsulation — a semantic property of the modeled system, and exactly what makes an interface an interface.
2. **Transformer.** The transformation tool sits *outside* the modeled world and is given full visibility: every agent, every nested dispatch, and all metadata. It needs this to generate an agent's implementation body.
3. **Human reading the visualization.** A third audience, whose view determines whether the picture is a faithful source of truth.

The encapsulation boundary is not just a constraint — it is **signal**. Dispatches that cross a session boundary are the target agent's public interface; the nested dispatches the caller cannot see are its private body. That boundary is precisely what tells a transformer what to emit as public vs. private, and what belongs in a protocol/interface vs. a concrete type.

---

## 5. Determinism as a per-agent dial

Determinism is not a property the whole system must have; it is an attribute set per agent. Control-flow logic can be supplied as a serialized code path, as pseudocode, or as a natural-language description — and the model can host a hybrid: mostly deterministic flows with islands of deliberate fuzziness.

This enables mixing deterministic and AI/"fuzzy" work, with the granularity and blast radius controlled by how agents are defined — e.g. ten deterministic agents for critical features and one fuzzy agent for prototyping. The emitted code may differ between runs, which is acceptable whenever the nature of the work allows it (specs, prototypes, exploration).

**This is gradual typing.** The interesting engineering lives at the *boundary* where a fuzzy agent's output crosses into a deterministic consumer. Fuzziness propagates downstream unless stopped, so the contract belongs on that edge, not inside either agent. The `params_schema` is that contract — but it must be **enforced** at the boundary (validate and coerce before the deterministic agent trusts the value), not merely declared. The "blame" discipline from gradual-typing theory applies: track which side of a boundary violated the contract, so a fuzzy node is blamed rather than the deterministic node that received its bad output. That tracking is what stops one fuzzy agent from silently corrupting ten deterministic ones.

---

## 6. Reliability model

For any target that must be deterministic, reliability reduces to a single question:

> **Is each node fully specified, or does it contain a natural-language gap?**

Reliability is monotonic along the specification spectrum:

- **Serialized code path → output:** deterministic transcription; cannot be gotten wrong.
- **Pseudocode → output:** nearly deterministic.
- **Prose description → output:** an inference step with genuine degrees of freedom — where reliability leaks.

Natural-language nodes are appropriate when the *target* is itself prose (a tech or product spec) or when fuzziness is intended. They are simply not in the reliable subset when the target is deterministic code.

A second, related subtlety — **statement vs. execution.** A program and an execution trace are different objects: the program generates all traces; a trace is one path. If dispatches represent *statements* (including `if`/`else` and loops as first-class nodes), the tree is effectively an AST and generation is transcription. If dispatches represent *executed operations* on a single run, control flow is underdetermined: a conditional shows only the branch that fired, and a loop that ran three times is indistinguishable from three statements without a hint. This model targets the statement interpretation, with control flow represented explicitly rather than inferred from examples.

---

## 7. The abstraction ladder runs both ways

Sessions are *more concise than code* for the orchestration/interaction tier — async flows, protocols, lifecycle, who-calls-whom — and there they are a genuine step **up** the ladder. For computation-dense logic (a numeric routine, a parser, a tight reduction), expressing the work as message-passing between cell-agents is *more verbose* than the code, a step **down**.

Practical consequence: the model is an excellent top layer and a clumsy bottom one, so the art is knowing where to stop decomposing. The per-variable-agent capability is always available, so a naive author or transformer can over-apply it and produce pathologically granular graphs for logic a single expression would have stated better. When a complex algorithm needs it, an agent can still carry a deterministic code implementation directly.

Importantly, **sessions define behavior but do not run inside the application.** Using the actor model as the live runtime driver is not feasible given interaction complexity and round-trip latency; it is a design-time modeling and specification layer that compiles down and out.

---

## 8. Agent analysis: architecture as a measurable search problem

Because behavior is pinned by the interactions and the decomposition is a free variable, **architecture becomes a search problem with correctness held constant.** Questions normally settled by judgment become studyable against the same fixed behavior:

- How small or large should an agent be?
- How many ops should an agent support?
- When should an op be promoted into its own agent?
- Which agent boundaries make the most sense?
- How can the number of messages passed be reduced?

This is new leverage: coupling and cohesion stop being rhetoric and become measurable, because interactions are first-class data rather than implicit in code, and competing decompositions of the *same* trace can be enumerated and scored.

**Caution — do not optimize message count alone.** Minimizing inter-agent traffic is a degenerate objective whose global optimum is a single god-agent with zero messages, i.e. the worst design. Coupling cannot be minimized in isolation; a counter-pressure (cohesion, or a per-agent complexity penalty) is required, or the optimizer collapses everything into one blob. The healthy objective is two-sided — the classic coupling-vs-cohesion tension, now measurable rather than merely argued.

---

## 9. Transformers

A transformer consumes the full session graph (plus metadata) and emits an artifact: code in some language, a tech spec, a product spec. Multiple transformers may exist at different levels of conciseness.

Two properties follow from full transformer visibility:

- **It can partition output correctly** using the encapsulation boundaries as signal (public interface vs. private body).
- **Structural fidelity is a policy, not a guarantee.** Because the transformer sees through the boundaries and is not bound by them, whether the emitted *code* preserves the modeled agent boundaries is its choice. An optimizing transformer may inline private dispatches, collapse a cell-agent into a local, or dissolve modeled boundaries — fine for performance and consistent with not needing a one-to-one mapping. The in-model encapsulation pins the *interface* a caller codes against, not the *structure* of what is emitted behind it. If the emitted module structure must mirror the session structure (for debuggability, or so the visualized thing matches the steppable thing), that is a constraint handed explicitly to the transformer.

---

## 10. Theoretical lineage

The model independently lands on well-explored foundations, which is evidence it coheres:

- **Actor model.** Agents are actors; message-passing is the only interaction; state is encapsulated and reachable only through ops; no shared memory. "One agent per parameter with set/get" is the actors-as-mutable-cells construction. "Pass agents as arguments" is higher-order actors / channel mobility — the ingredient that makes such a model composable and general.
- **Observable / trace semantics and process calculi.** Treating anything that does not cross an agent boundary as unobservable mirrors the stance that two programs with the same interactions are the same program.
- **Session types.** The natural next step for provable reliability: type the *protocol* — the legal order and shape of the whole exchange — not just each message's inputs. The `params_schema` types inputs; typing the sequence lets a checker reject a malformed interaction *before* any transformer runs. This is the difference between reliability you can trust and reliability you hope for.
- **Gradual typing and blame.** The framework for mixing deterministic and fuzzy regions with contracts and fault attribution at the boundary (see §5).
- **Dependency injection.** Falls directly out of coupling agents only to their message contracts (see §2, agent-as-argument).

---

## 11. Open design decisions to pin down

- **Cell-agent consistency model.** Modeling a variable as a set/get agent can introduce concurrency and aliasing the original code never had: two sessions holding the same cell-agent and both calling `set` raise an ordering question. Decide explicitly whether cell-agents are single-assignment, sequentially consistent, or looser. By the model's own "observable interactions" axiom, ordering that changes observable behavior is in-scope and cannot be fully hidden; ordering that does not (memory layout, calling conventions) can be relegated to a hidden layer.
- **Governance layer visibility.** A separate rules/governance layer may carry fine-grained behavioral specs. The honest rule: anything in it that affects *observable* ordering must surface in the visualization, or two visually identical graphs could behave differently and the "understand the system by looking at the sessions" promise acquires an asterisk.
- **Control-flow representation.** Confirm that branches and loops are first-class, explicitly represented nodes (statement interpretation) rather than artifacts of a single execution trace (see §6).
- **Boundary enforcement.** Decide where `params_schema` contracts are checked at runtime vs. assumed, especially on fuzzy→deterministic edges.

---

## 12. Worked shape (illustrative)

A `View` initializing through a `ViewModel`:

- `View` opens a session with `ViewModel` and sends `init(arg)`.
- Inside that session, `ViewModel` performs nested dispatches the `View` cannot see — e.g. a session with an `APIService` agent to fetch, and set-ops against the parameter agents it owns.
- `ViewModel` returns a response to `View`, closing the `init` session.

From this, a transformer with full visibility can recover: the public signature `init(arg)` on the ViewModel (from the visible boundary), the private body (from the nested dispatches), the dependency on `APIService` (from the inner session), and the wiring (from agents passed as arguments). What it cannot recover without explicit specification: branch/loop structure not represented as nodes, local computation never expressed as a dispatch, and the concrete types of intermediate values not carried in payloads.

---

*This document captures a model under active design. The reliable subset is whatever is specified down to a level where the transformer never has to choose; everything else is a deliberate, scoped choice to allow fuzziness.*
