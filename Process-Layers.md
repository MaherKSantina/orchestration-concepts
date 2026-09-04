# Code vs Graph in a Layered Process System
## A compact blueprint for process representation, synthesis, and execution

## Purpose

This document defines how **code**, **graphs**, and **process layers** fit together in a left-system architecture.

It is not a comparison of technologies. It is a model for deciding:
- what should be represented explicitly
- what may be generated
- what should execute locally
- what should remain analyzable
- where code is appropriate
- where graphs are necessary

The goal is to preserve a **first-class process system** while still using code and AI where they provide leverage.

---

## Core one-liners

- **Use data to describe the machine. Use code to power the parts.**
- **The real distinction is not code vs graph. It is generator vs generated artifact.**
- **Code is not only execution; it can also be synthesis.**
- **The process definition must be representable as explicit topology.**
- **Computation should be confined to step-local transformers or process synthesis.**
- **The canonical artifact should be the explicit process definition, not the generator.**
- **Graphs represent orchestration structure. Code represents bounded local behavior or generative rules.**
- **A left system should externalize process knowledge as explicit artifacts, even when those artifacts are produced by code or AI.**

---

## 1. Representation vs computation

**One-liner:**  
**Data describes a bounded structure. Code generates behavior.**

The main distinction is not:
- JSON vs code
- graph vs AST
- visual vs textual

The deeper distinction is:
- **representation**: what the system *is*
- **computation**: how the system *behaves*

A process graph is primarily a **representation**.  
Code is primarily a **computation**.

A representation tends to be:
- explicit
- bounded
- serializable
- inspectable
- storable
- queryable

A computation tends to be:
- generative
- conditional
- procedural
- potentially unbounded
- more expressive
- harder to normalize semantically

Another useful distinction is:
- **closed representation**: a finite process object is explicitly described
- **open computation**: a procedure can generate many possible manifestations

This is why code feels more powerful and more slippery at the same time.

---

## 2. Code and graph are not opposites

**One-liner:**  
**They become incompatible only when they compete to be the same source of truth.**

They optimize for different things.

**Graph / encoded process language** optimizes for:
- visibility
- analysis
- process diffing
- versioning
- governance
- topology inspection
- reusable orchestration semantics

**Code** optimizes for:
- expressive power
- abstraction
- dynamic behavior
- general computation
- local reuse
- fast implementation of complex logic

Two logically equivalent processes can be written in very different code forms:
- different helper structure
- different control flow
- dynamic generation
- hidden imports
- loops
- callbacks
- higher-order wrappers

ASTs help with syntax and local structure.  
They do not automatically expose the normalized process meaning needed for:
- graph diffing
- topology comparison
- path analysis
- discovery promotion
- explicit hierarchy
- process-level versioning

A good fit:
- code generates or executes
- graph remains the explicit orchestration artifact

A bad fit:
- code is the only place where orchestration exists
- no explicit process artifact is preserved

---

## 3. The process layer stack

**One-liner:**  
**The useful hierarchy is implicit process -> process synthesis -> explicit orchestration -> execution.**

The useful stack is:

1. **Implicit layer**
2. **Process synthesis layer**
3. **Explicit orchestration layer**
4. **Execution layer**

This hierarchy is more important than the code-vs-graph distinction.

### 3.1 Implicit layer
**One-liner:**  
**Implicit process knowledge is real process logic that has not yet been externalized.**

This layer lives in people’s minds or other non-formalized reasoning spaces.

It includes:
- choosing what kind of process is needed
- recognizing a new case shape
- deciding which variables matter
- selecting the next process manifestation
- judgment that has not yet become an artifact

The left system exists partly to move useful parts of this layer upward into explicit artifacts.

### 3.2 Process synthesis layer
**One-liner:**  
**A meta-process can generate a bounded process.**

This layer produces process definitions rather than directly solving the business task.

Its input may be:
- parameters
- templates
- context bundles
- higher-order rules
- AI-generated proposals
- human instructions

Its output should be:
- a bounded process definition
- with explicit steps
- explicit contracts
- explicit routing
- explicit metadata

This layer may be implemented by:
- code
- AI
- rules
- template systems
- human authoring

### 3.3 Explicit orchestration layer
**One-liner:**  
**The process definition must be representable as explicit topology.**

This is the core process artifact that should be:
- stored
- diffed
- versioned
- analyzed
- executed
- audited

This layer defines:
- steps
- edges
- routing
- contracts
- states
- policies
- metadata
- child-process relationships

### 3.4 Execution layer
**One-liner:**  
**Computation should be confined to step-local transformers.**

This layer runs individual steps.

A step implementation may be:
- code
- AI
- human
- external tool
- deterministic function

The execution layer should stay:
- local
- bounded
- observable
- contract-driven
- replaceable

---

## 4. The three roles of code

**One-liner:**  
**Code can execute, synthesize, or dominate. Only the third is dangerous.**

Code can appear in three roles:

### 4.1 Code as execution
A function performs one step with a bounded contract.

This is desirable.

### 4.2 Code as synthesis
A function generates a bounded process definition.

Examples:
- generating a process graph from input parameters
- expanding a reusable pattern into a bounded workflow
- compiling a DSL or template into a process definition

This is also desirable, provided the generated process is preserved as an explicit artifact.

### 4.3 Code as orchestration source of truth
Code directly encodes the orchestration with no separate explicit process artifact.

This is the dangerous case for a left system because:
- process semantics stay compressed inside procedure
- graph-level analysis becomes harder
- process diffing becomes weaker
- reusable process formalization becomes weaker
- discovery-to-formalization becomes less legible

---

## 5. The canonical artifact rule

**One-liner:**  
**The canonical artifact should be the generated process, not only the generator.**

Even if a process is created by:
- code
- AI
- templates
- a DSL compiler
- a human

the explicit **process definition** should usually be what the system treats as canonical for:
- storage
- versioning
- execution
- audit
- analysis
- comparison

The generator still matters, but it is a different artifact.

This distinction lets the system preserve both:
- how the process was produced
- what process was actually produced

---

## 6. Topology vs transformer

**One-liner:**  
**Graphs expose orchestration structure. Transformers perform local work.**

A useful distinction:
- **topology**: how the process is structured
- **transformer**: how a step maps input to output

Orchestration is about topology.  
Execution is about transformers.

This is why code fits well at the execution layer: it can act as a bounded transformer.

A graph does not need to express every possible infinite behavior.  
It needs to express the bounded orchestration shape that the system will store and analyze.

Some behavior is naturally generative:
- formulas
- synthesis rules
- parameterized expansions
- reusable compilation logic
- local algorithmic transformations

This is fine.

The requirement is not:
- “everything must be a graph”

The requirement is:
- “the orchestration artifact must become explicit at the level where analysis and governance matter”

---

## 7. Why code-only orchestration weakens higher-order features

**One-liner:**  
**If orchestration exists only in code, the process stops being a first-class object.**

When orchestration is code-only, the system struggles to support:
- process-definition diffing
- process mining across definitions
- topology analytics
- bounded discovery formalization
- promotion of emerging processes
- process-level comparison
- graph-native editing
- explicit process lineage

The problem is not that code cannot express behavior.  
The problem is that the orchestration meaning is no longer exposed in a stable, bounded, queryable form.

---

## 8. Design rules for a left system

**One-liner:**  
**Use data to describe the machine. Use code to power the parts.**

### 8.1 Orchestration rule
**Process knowledge should be externalized as explicit artifacts.**

### 8.2 Code rule
**Use code for bounded execution or bounded synthesis, not as the sole owner of orchestration semantics.**

### 8.3 Preservation rule
**If code or AI generates a process, preserve the generated process as a first-class artifact.**

### 8.4 Analysis rule
**Higher-order features should analyze the explicit process definition, not reconstruct semantics from implementation code whenever possible.**

### 8.5 Layering rule
**Do not collapse synthesis, orchestration, and execution into one layer.**

---

## 9. Minimal entity model

A compact conceptual model:

### ProcessSynthesizer
Produces a process definition from inputs or rules.

### SynthesisInput
The parameters or context used to generate a process.

### SynthesisTrace
Why a particular process shape was produced.

### ProcessDefinition
The explicit, bounded orchestration artifact.

### StepDefinition
A step within the process definition with a stable contract.

### ExecutionHandler
The implementation used to run a step.

### ProcessInstance
A run of a process definition.

This model preserves:
- higher-order generation
- explicit process representation
- local execution

without collapsing them into one concept.

---

## 10. Practical blueprint

### What should be represented explicitly
- process topology
- step identities
- step contracts
- routing
- process metadata
- process hierarchy
- process version
- orchestration policies

### What may be generated
- process definitions
- subgraphs
- bounded variants of a process
- reusable process templates

### What may be procedural
- step-local logic
- local transformations
- calls to tools
- calls to AI
- deterministic helper logic

### What should remain analyzable
- generated process definitions
- process diffs
- process lineage
- process versions
- common paths
- edge-case paths
- promotion candidates

---

## 11. Final synthesis

**One-liner:**  
**Let code and AI generate orchestration when useful, but preserve the resulting orchestration as an explicit, bounded, analyzable artifact.**

The architecture is not:
- code or graph

It is:
- implicit process knowledge
- process synthesis
- explicit orchestration
- bounded execution

This lets the system:
- use code productively
- use AI productively
- preserve analyzability
- preserve process semantics
- support higher-order features later
