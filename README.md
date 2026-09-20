# Orchestration Concepts

Working notes and concept references from building a personal agent-orchestration platform — a system I use daily to run real work through AI agents, deterministic tools, and explicit processes. The platform itself is React, TypeScript, Node and Postgres, with Claude Code as the daily driver; these documents are the ideas that kept proving out along the way.

The through-line: **AI didn't replace the system — it became one agent inside it.** Most of what's here is about the machinery around the model: governance, process formalization, supervision, and making implicit work explicit.

## Start here

- **[A Governed Agent Architecture](governed-agent-architecture.md)** — forced deliberation, auditable execution, and externalized reasoning. Not a faster agent or a more autonomous one: an agent designed to make implicit work explicit, expensive decisions auditable, and routine decisions cheap.
- **[Left System vs Right System](Left-System-vs-Right-System.md)** — when work deserves a formalized process (typed contracts, evaluators, human review points) versus a context-driven agent. The boundary-drawing model everything else hangs off. There's an interactive [visualization](left_right_system_visualization.html), and a full [functional specification](Left-System-Functional-Spec.md) of the left system.
- **[Observation / Execution Model](Observation%20Execution%20Model.md)** — a dual-layer model where a supervision process watches running executions and decides to let them continue or intervene.

## The rest

- **[Agent Sessions as a Behavioral Model](agent-sessions-model.md)** — interactions between agents as the primary artifact; code, tech specs and product specs become compilation targets from the same session graph.
- **[Process Layers: Code vs Graph](Process-Layers.md)** — use data to describe the machine, use code to power the parts; why the canonical artifact is the explicit process definition, not the generator.
- **[Checkpoint–Artifact Model](Checkpoint-Artifact%20Model.md)** — identity plus historical change as the only stored state; everything else derived by replay.
- **[Task-Graph Model](Task-Graph%20Model.md)** — hierarchical task planning with reusable templates and grouped alternative strategies.
- **[Orchestrator: Interaction-First Specification](orchestrator-interaction-first-spec.md)** — start from real interactions, externalize recurring structure, promote it into explicit process definitions.
- **[Abstraction and Function Creation Methodology](Abstraction%20and%20Function%20Creation%20Methodology.md)** — a domain-agnostic method for isolating controllable variables and extracting reusable functions.
- **[Methodology Machinery](methodology_machinery.md)** — the same machinery applied to different sources of truth (code-as-source vs spec-as-source).
- **[platform-specs/](platform-specs)** — the product and technical specifications the platform was actually built from.
- **[Claude configuration use cases](claude-dimensions/claude-dimensions.playbook)** — eight things to do with Claude's configuration (see what a session reads, see its memory, keep a memory of your own, guard what is written, configure one part, promote to project, team or org) as playbook events, each a brief with a Claude Code item and a chat app item.

## Context

These are living documents written for my own use first — the register is reference material, not blog posts. They describe a single-operator system, deliberately: one person orchestrating agents across their own work is the laboratory where these concepts get tested every day.
