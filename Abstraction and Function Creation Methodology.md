# Abstraction and Function Creation Methodology
## Version 2 — Formal Reference

## 1. Purpose

This document defines a general methodology for producing abstractions, discovering functions, and composing reusable systems across domains.

It is intended for work where the goal is not only to apply existing frameworks, but to:
- identify meaningful differences
- isolate controllable variables
- extract reusable functions
- map operating boundaries
- compose higher-order methods or systems

This methodology is domain-agnostic. It can be applied to music, workflows, software systems, product design, writing, strategy, and other areas where discovery and judgment interact.

---

## 2. Objective

The objective is to convert raw perception and experimentation into reusable knowledge.

That conversion follows this progression:

**perception -> variation -> effect -> function -> boundary -> composition**

The output is not a set of examples. The output is a set of:
- functions
- rules
- boundary maps
- compositions
- reusable decision structures

---

## 3. Operating Principles

### 3.1 Primary Principle
**Do not collect examples. Extract functions with boundaries so new examples can be generated deliberately.**

### 3.2 Core Structural Principle
**AI at the edges, logic in the core.**

This means:
- AI may assist with translation, generation, summarization, coding, and visualization
- explicit logic should govern repeatable execution
- human cognition remains responsible for abstraction, judgment, and function discovery

### 3.3 Discovery Principle
**Structured variation under evaluation is the engine of abstraction.**

### 3.4 Control Principle
A useful method should optimize for:
- explicitness
- repeatability
- testability
- controllability
- bounded assumptions
- incremental extensibility

### 3.5 Interpretation Principle
**Tools accelerate variation and comparison. Mind extracts function.**

---

## 4. Scope

This methodology applies when:
- the problem space contains meaningful variation
- value depends on understanding why something works
- repeated experimentation is possible
- functions need to be reusable beyond a single example

This methodology is less useful when:
- the task is purely mechanical and already fully specified
- there is no meaningful evaluation criterion
- the output does not need to generalize
- the work is entirely one-off and non-transferable

---

## 5. Key Terms

## 5.1 Taste
The initial sense that something is valuable, effective, beautiful, elegant, wrong, or weak.

Taste is the starting sensor. It is subjective, but still operationally useful.

Taste may be divided into:
- **personal taste**: what feels right to the individual
- **shared taste**: what repeatedly lands well with others in context
- **functional fit**: what serves the objective even if it is not preferred

## 5.2 Contrast
A meaningful difference that triggers investigation.

Contrast is the signal that there may be a discoverable function.

Examples:
- one drop feels larger than another
- one workflow feels more reliable than another
- one interface feels clearer than another

## 5.3 Variable
A controllable lever whose change may influence the outcome.

A variable is not merely a difference. It is a manipulable dimension.

## 5.4 Effect
A change in the perceived or measured outcome resulting from manipulation of one or more variables.

## 5.5 Function
The role performed by a variable or pattern under certain conditions.

A function is not the implementation itself. It is the reusable role the implementation performs.

## 5.6 Boundary
The limit of a function’s usefulness, validity, or effectiveness.

Boundaries define:
- where the function works
- where it degrades
- where it fails
- what conditions are required

## 5.7 Composition
The deliberate combination of multiple functions into a larger pattern, method, style, framework, or system.

## 5.8 Interface
The means by which the exploration space is manipulated and observed.

Examples:
- encoders
- mapped controls
- macros
- toggles
- scripts
- dashboards
- parameter panels

Interface quality affects exploration speed and quality.

## 5.9 Primitive
A lower-level mechanism, constraint, or building block from which local effects arise.

## 5.10 Mechanism
The explanation of how or why a variable produces a given effect.

---

## 6. Core Model

The methodology is grounded in the following analytical model:

- **taste** identifies what feels valuable
- **contrast** identifies what seems meaningfully different
- **variable** identifies what changed
- **effect** identifies what changed in the outcome
- **function** identifies what role is being performed
- **boundary** identifies where that role stops holding
- **composition** identifies how functions combine
- **interface** determines how efficiently the space can be explored

This model is recursive. A composition at one level may become a variable or primitive at a higher level.

---

## 7. Discovery Lifecycle

The methodology is best treated as a state-based loop rather than a one-way linear sequence.

The main states are:
1. exploration
2. abstraction
3. boundary testing
4. composition

Progression occurs when the current state has produced a stable enough claim to justify the next type of work.

---

## 8. State Definitions

## 8.1 Exploration

### Goal
Surface meaningful differences and candidate patterns.

### Driving Question
What seems to matter?

### Typical Activities
- observing examples
- making controlled changes
- comparing outcomes
- noticing repeated effects
- collecting candidate contrasts

### Output
- candidate variables
- candidate effects
- candidate relationships

### Exit Condition
Move to abstraction when a repeatable pattern is noticeable and can be stated as a candidate relationship between a variable and an effect.

### Minimum Claim Required
A minimal exploration output should support a sentence of this form:

**Changing X appears to affect Y.**

---

## 8.2 Abstraction

### Goal
Convert repeated effects into candidate functions.

### Driving Question
What is this doing?

### Typical Activities
- isolating variables
- naming effects
- proposing mechanisms
- compressing observations into reusable claims

### Output
- candidate functions
- candidate mechanisms
- concise explanatory statements

### Exit Condition
Move to boundary testing when the function can be expressed as a falsifiable claim.

### Minimum Claim Required
A minimal abstraction output should support a sentence of this form:

**Changing X tends to produce Y under Z conditions.**

---

## 8.3 Boundary Testing

### Goal
Determine the operating range and failure modes of the function.

### Driving Question
When does this hold, weaken, or fail?

### Typical Activities
- parameter sweeps
- edge-case testing
- toggling conditions
- stress testing
- comparison against opposite cases

### Output
- operating conditions
- failure modes
- tradeoffs
- acceptable ranges

### Exit Condition
Move to composition when the main boundaries are sufficiently understood for the intended use.

### Stopping Rule
Boundary testing ends at **practical saturation**, not theoretical completeness.

Practical saturation is reached when:
- the primary operating range is known
- obvious failure modes are known
- repeated tests produce little new insight
- predictions become reliable enough for use

---

## 8.4 Composition

### Goal
Use validated functions as building blocks in higher-order systems.

### Driving Question
What combines well with this, and toward what larger effect?

### Typical Activities
- combining functions
- sequencing functions
- resolving tradeoffs between functions
- building frameworks, styles, or systems

### Output
- reusable methods
- frameworks
- patterns
- architectures
- styles

### Exit Condition
Composition is mature when the function is deliberately reusable as part of a larger whole.

---

## 9. Transition Logic

Move to the next state when the uncertainty changes form.

- In **exploration**, uncertainty is: **what is happening**
- In **abstraction**, uncertainty is: **what is this doing**
- In **boundary testing**, uncertainty is: **when does this hold**
- In **composition**, uncertainty is: **how does this interact with other things**

This is the generic state transition rule.

---

## 10. Practical Progress Signals

A state is often ready to advance when the following signals appear.

## 10.1 Novelty Drops
New experiments stop producing new categories of insight.

## 10.2 Compression Improves
The pattern can be described more concisely without losing explanatory power.

## 10.3 Prediction Improves
Future outcomes become increasingly predictable before testing.

## 10.4 Transfer Appears
The same function begins to appear across multiple cases.

---

## 11. Producer vs Consumer Modes

## 11.1 Consumer Mode
In consumer mode, the actor starts with a preexisting function library and applies named functions to new situations.

This mode assumes:
- the function taxonomy already exists
- the abstractions are accepted
- the task is mostly application and adaptation

## 11.2 Producer Mode
In producer mode, the actor begins from repeated effects and extracts new functions.

This mode requires:
- detecting contrasts before formal language exists
- isolating candidate variables
- building explanatory claims
- discovering boundaries before standardization
- naming functions after evidence, not before it

### Producer’s Rule
**Start from repeated effects, not from pre-named functions.**

---

## 12. Function Creation Process

Function creation proceeds from lower-level structure toward higher-level reuse.

The general stack is:

**primitives -> local effects -> functions -> compositions**

## 12.1 Primitive
A low-level mechanism, lever, or constraint.

## 12.2 Local Effect
The directly observed outcome of manipulating a primitive.

## 12.3 Function
The generalized role served by the effect.

## 12.4 Composition
The larger structure created by combining multiple functions.

### Example Pattern
- primitive: reduce density before impact
- local effect: stronger contrast at the impact moment
- function: amplify perceived release
- composition: combine with tension build and transient emphasis

---

## 13. Minimal Function Formula

A function should be expressible in a reusable form.

### Minimum Form
**Changing X tends to produce Y under Z conditions.**

### Stronger Form
**When X is changed, under Y conditions, Z effect tends to occur because M mechanism becomes more influential.**

This formula is preferred because it forces:
- variable identification
- condition identification
- effect identification
- mechanism hypothesis

---

## 14. Function Naming Rules

A function name should:
- describe the role, not merely the implementation
- remain reusable across similar cases
- avoid overfitting to one example
- be grounded in repeated effects and boundaries

Naming should generally happen after:
1. the effect is stable enough
2. the variable is legible
3. the mechanism is plausible
4. the boundary is at least partially mapped

Premature naming produces vague frameworks. Delayed naming produces repeated rediscovery.

---

## 15. Boundary Taxonomy

Boundary mapping should distinguish at least three types.

## 15.1 Hard Boundaries
The result objectively breaks or becomes invalid.

Examples:
- the command cannot execute
- the system no longer functions
- the outcome collapses into failure

## 15.2 Soft Boundaries
The result still works but quality decreases.

Examples:
- weaker impact
- reduced clarity
- less coherence
- slower execution
- higher friction

## 15.3 Social or Cultural Boundaries
The result is technically valid but is rejected by norms, context, audience expectation, or group taste.

Examples:
- a sound fits technically but not stylistically
- a workflow is correct but socially unworkable
- a process is efficient but unacceptable to stakeholders

---

## 16. Variable Vocabulary Development

Progress depends on learning the controllable levers of the domain.

For any domain, build a variable vocabulary by asking:
- what are the main levers?
- what values can they take?
- which variables dominate outcomes?
- which variables interact strongly?
- which are foundational and which are decorative?

### Example Domains

#### Music
- density
- width
- transient sharpness
- frequency balance
- tension duration
- repetition rate
- layering
- timing offsets
- space
- contrast

#### Workflows and Systems
- number of handoffs
- explicitness of required context
- number of allowed states
- retry logic
- dependency structure
- visibility of progress
- validation strictness
- coupling
- ownership clarity

A domain becomes easier to abstract when its variable vocabulary becomes explicit.

---

## 17. Role of Perception

Perception is not optional. It is the starting condition of the method.

However, perception has limitations:
- the senses may be undertrained
- relevant contrast may be hidden without tools
- norms may normalize brokenness
- the actor may detect a real issue before the group has language for it

Accordingly, perception must be supported by:
- comparison
- instrumentation
- repeated exposure
- deliberate listening, observation, or review
- contrast-oriented analysis

---

## 18. Role of Tooling and Automation

Automation is most useful when it accelerates:
- variation
- parameter sweeps
- toggling
- repeated execution
- boundary testing
- structured validation
- artifact generation
- result capture

Automation is less suitable for:
- deciding what is valuable
- discovering meaning without interpretation
- creating trustworthy logic from hidden assumptions
- replacing function extraction itself

### Automation Principle
**Automate variation and evaluation support before attempting to automate interpretation.**

### Interface Principle
A better exploration interface shortens the loop from:
**observation -> variation -> comparison -> conclusion**

---

## 19. Role of AI

AI should be positioned as an assisting component rather than the primary driver of deterministic systems.

## 19.1 Appropriate Uses
- translating natural language into structured input
- drafting code and tooling
- summarizing results
- generating test cases
- visualizing outputs
- proposing hypotheses
- accelerating boilerplate and scaffolding

## 19.2 Inappropriate Uses
- acting as the system of record for repeatable logic
- making silent assumptions in critical execution
- being trusted as the core deterministic mechanism
- substituting for explicit validation

### AI Operating Principle
**AI proposes. Logic decides. Tools execute. Validation verifies.**

---

## 20. Structured Input Principle

Natural language is useful for intake and rough intent, but should not be the long-term system of record for repeatable logic.

Prefer structured representations such as:
- forms
- typed objects
- enums
- JSON
- explicit commands
- diagrams
- schemas

Natural language may remain useful at the edges, especially for:
- ambiguous input
- initial translation
- explanation
- summarization

For scalable determinism, structure must eventually dominate.

---

## 21. Common Failure Modes

## 21.1 Abstracting Too Early
A single case is treated as a general function.

## 21.2 Varying Too Many Variables at Once
Attribution becomes weak and learning becomes noisy.

## 21.3 Confusing Taste with Mechanism
Value is sensed, but the role being performed is not isolated.

## 21.4 Naming Before Boundary Mapping
The abstraction sounds coherent but is fragile in practice.

## 21.5 Seeking Total Certainty Before Progressing
The method stalls because candidate claims are never advanced to testing.

## 21.6 Over-Automating Interpretation
Tooling attempts to replace the highest-value cognitive step before stable structure exists.

---

## 22. Progress Criteria

Progress is being made when at least one of the following occurs:
- a previously hidden contrast becomes legible
- a new variable becomes controllable
- a repeated effect becomes reproducible
- a mechanism becomes plausible
- a boundary becomes identifiable
- a function becomes nameable
- the function transfers across cases
- the function composes into a larger reusable system

A small, real function is more valuable than a large, vague framework.

---

## 23. Discovery Checklist

Use the following questions to drive function production:

1. What did I notice?
2. What contrast triggered investigation?
3. What changed?
4. What effect changed?
5. Under what conditions?
6. Does this pattern repeat?
7. What mechanism might explain it?
8. Where does it stop helping?
9. Where does it fail?
10. Is the function reusable?
11. What could it compose with?
12. What should it be called?

---

## 24. Standard Capture Template

Use the following structure to capture discoveries consistently.

### Observation
What was noticed?

### Contrast
What seemed meaningfully different?

### Variable
What controllable lever changed?

### Manipulation
What was actually altered?

### Effect
What changed in the outcome?

### Candidate Function
What role may this be performing?

### Mechanism
Why might this produce the effect?

### Conditions
What must be true for the claim to hold?

### Boundary
Where does the function weaken, stop helping, or fail?

### Reusable Rule
How can this be deliberately applied again?

### Composition
What other functions combine well with it?

### Confidence
How stable is the current claim?

### Next Test
What should be varied next?

---

## 25. Example: Music

### Observation
A particular drop feels larger and more satisfying.

### Contrast
Why does this drop feel larger than another similar one?

### Variable
There is less happening before the impact, and width opens after the hit.

### Effect
The release feels stronger and more spacious.

### Candidate Function
Contrast in density and width amplifies perceived release.

### Mechanism
Reduced pre-impact density increases contrast salience, while post-impact width enlarges the perceived field.

### Conditions
The rhythmic expectation must already be established.

### Boundary
Too much emptiness weakens momentum. Too much width weakens focus.

### Reusable Rule
Reduce pre-impact density when stronger release is needed, but preserve enough motion to maintain expectation.

### Composition
Combine with tension build, low-end restraint, and transient emphasis.

---

## 26. Example: Workflow Design

### Observation
One workflow fails less often than another.

### Contrast
Why is one workflow more reliable despite serving the same purpose?

### Variable
The more reliable workflow forces explicit required input before execution.

### Effect
Errors appear earlier and downstream ambiguity decreases.

### Candidate Function
Structured input exposes missing information earlier.

### Mechanism
Required fields force clarification before later decisions compound hidden assumptions.

### Conditions
The task must be sufficiently repeatable for structure to be worthwhile.

### Boundary
Too much rigidity blocks exploration and low-certainty work.

### Reusable Rule
Use strict structure for repeatable deterministic flows; relax structure for exploratory flows.

### Composition
Combine with explicit states, deterministic routing, and validation before execution.

---

## 27. Condensed Operational Summary

### Discovery Engine
**Structured variation under evaluation**

### Core States
1. exploration
2. abstraction
3. boundary testing
4. composition

### Core Analytical Model
- taste
- contrast
- variable
- effect
- function
- boundary
- composition
- interface

### Producer’s Rule
**Start from repeated effects, not from named functions.**

### Function Formula
**Changing X tends to produce Y under Z conditions.**

### Execution Principle
**Tools accelerate variation and comparison. Mind extracts function.**

### AI Principle
**AI proposes. Logic decides. Tools execute. Validation verifies.**

---

## 28. Final Position

The purpose of this methodology is not merely to consume frameworks more effectively. It is to create a repeatable path for discovering reusable functions, understanding their limits, and composing them into larger systems.

That is the shift from consumer to producer.

A producer of abstractions turns:
- sensed differences
- controlled variation
- observed effects
- boundary knowledge

into:
- functions
- rules
- compositions
- systems
- new ways of working
