# Pragmatism

A standalone review guide for judging whether software is useful, changeable, safe, and operable. Pragmatism is not permission to skip rigor: match effort to risk, preserve feedback, and prefer reversible evidence-backed decisions over ceremony or speculation.

## Synthesis

- Start with the user outcome and current constraints; distinguish requirements from assumptions.
- Keep changes easy to reverse and components independently changeable. Remove duplication of knowledge, not merely similar text.
- Work in thin end-to-end slices so integration, performance, security, and operability are tested early.
- Make contracts, ownership, failure modes, and resource limits explicit. Fail early at trust boundaries and recover deliberately.
- Treat tests, debugging evidence, version control, automation, and production signals as one feedback system.
- Reassess as evidence changes: refactor continuously, communicate tradeoffs, and own the result rather than the process.

## Review sequence

1. **Outcome:** What user or business result must change, and how will anyone know?
2. **Reality:** Which constraints and assumptions are verified? Is a prototype or tracer slice needed?
3. **Design:** Can responsibilities change independently, and is each dependency or abstraction earning its cost?
4. **Safety:** Are contracts, validation, security, concurrency, resources, and failure recovery explicit?
5. **Feedback:** Do focused tests and observable behavior cover the highest risks, including integration paths?
6. **Delivery:** Is the change reversible, incrementally releasable, documented where needed, and owned after release?
7. **Simplify:** Remove speculative flexibility, duplicated knowledge, stale code, and process that does not improve the outcome.

## Topic index

### Engineering mindset

- [Agency](a-pragmatic-philosophy.md#agency)
- [Responsibility](a-pragmatic-philosophy.md#responsibility)
- [Software entropy](a-pragmatic-philosophy.md#software-entropy)
- [Catalyzing change](a-pragmatic-philosophy.md#catalyzing-change)
- [Good-enough software](a-pragmatic-philosophy.md#good-enough-software)
- [Continuous learning](a-pragmatic-philosophy.md#continuous-learning)
- [Communication](a-pragmatic-philosophy.md#communication)

### Design and delivery

- [Easy-to-change design](a-pragmatic-approach.md#easy-to-change-design)
- [Duplication](a-pragmatic-approach.md#duplication)
- [Orthogonality](a-pragmatic-approach.md#orthogonality)
- [Reversibility](a-pragmatic-approach.md#reversibility)
- [Tracer bullets](a-pragmatic-approach.md#tracer-bullets)
- [Prototypes](a-pragmatic-approach.md#prototypes)
- [Domain languages](a-pragmatic-approach.md#domain-languages)
- [Estimation](a-pragmatic-approach.md#estimation)

### Working tools

- [Plain text](the-basic-tools.md#plain-text)
- [Shell fluency](the-basic-tools.md#shell-fluency)
- [Editor fluency](the-basic-tools.md#editor-fluency)
- [Version control](the-basic-tools.md#version-control)
- [Debugging](the-basic-tools.md#debugging)
- [Text manipulation](the-basic-tools.md#text-manipulation)
- [Engineering journal](the-basic-tools.md#engineering-journal)

### Defensive engineering

- [Contracts](pragmatic-paranoia.md#contracts)
- [Fail fast](pragmatic-paranoia.md#fail-fast)
- [Assertions](pragmatic-paranoia.md#assertions)
- [Resource ownership](pragmatic-paranoia.md#resource-ownership)
- [Bounded steps](pragmatic-paranoia.md#bounded-steps)

### Adaptable architecture

- [Decoupling](bend-or-break.md#decoupling)
- [Events](bend-or-break.md#events)
- [Transformation pipelines](bend-or-break.md#transformation-pipelines)
- [Composition over inheritance](bend-or-break.md#composition-over-inheritance)
- [Configuration](bend-or-break.md#configuration)

### Concurrency

- [Temporal decoupling](concurrency.md#temporal-decoupling)
- [Shared state](concurrency.md#shared-state)
- [Actors and processes](concurrency.md#actors-and-processes)
- [Blackboard coordination](concurrency.md#blackboard-coordination)

### Coding practice

- [Developer intuition](while-you-are-coding.md#developer-intuition)
- [Intentional programming](while-you-are-coding.md#intentional-programming)
- [Algorithmic cost](while-you-are-coding.md#algorithmic-cost)
- [Refactoring](while-you-are-coding.md#refactoring)
- [Tests as feedback](while-you-are-coding.md#tests-as-feedback)
- [Property-based testing](while-you-are-coding.md#property-based-testing)
- [Security](while-you-are-coding.md#security)
- [Naming](while-you-are-coding.md#naming)

### Product discovery

- [Requirements](before-the-project.md#requirements)
- [Constraint solving](before-the-project.md#constraint-solving)
- [Collaboration](before-the-project.md#collaboration)
- [Agility](before-the-project.md#agility)

### Teams and outcomes

- [Pragmatic teams](pragmatic-projects.md#pragmatic-teams)
- [Context-sensitive process](pragmatic-projects.md#context-sensitive-process)
- [Delivery foundations](pragmatic-projects.md#delivery-foundations)
- [User delight](pragmatic-projects.md#user-delight)
- [Ownership](pragmatic-projects.md#ownership)
