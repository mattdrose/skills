---
name: beautiful-code
description: Guides agents to write, refactor, and review clear, maintainable code and software architecture. Use when implementing code, reviewing a diff, simplifying complex code, improving names or boundaries, preserving behavior during refactoring, evaluating tests or error handling, or choosing application architecture and enterprise patterns.
---

# Beautiful Code

Apply this skill while writing, refactoring, or reviewing code. The key to beautiful code is to drive everything by the API you're building. Weather a function, class, service, controller, or anything alike; think about the interface that others will use, write it to be beautiful, and work inwards from there.

## Workflow

1. Read [core principles](references/principles.md) for general implementation or review work.
2. Read [smells](references/code-quality/smells.md) for code cleanup.
3. Use [the review workflow](references/review-workflow.md) when assessing existing code or a diff.
4. Read only the relevant topic under `references/code-quality/` for local implementation concerns.
5. Read only the relevant topic under `references/refactoring/` for behavior-preserving structural changes.
6. For system design, start with the [architecture index](references/architecture/index.md), then load only the applicable catalog.

Treat guidance as prompts rather than laws. Patterns must answer an observed design pressure; do not add indirection merely because a named pattern exists.

## Reference map

| File                                                                                                               | What it is                                                                   | When to use it                                                            |
| ------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| [references/principles.md](references/principles.md)                                                               | Core design principles: API-driven design, clarity, boundaries, and judgment | Start of most implementation or review work                               |
| [references/review-workflow.md](references/review-workflow.md)                                                     | Step-by-step process for assessing existing code or a diff                   | Reviewing a PR, patch, or unfamiliar change                               |
| [references/code-quality/names.md](references/code-quality/names.md)                                               | Naming for domain purpose, precision, and searchability                      | Choosing or improving identifiers                                         |
| [references/code-quality/functions-and-control-flow.md](references/code-quality/functions-and-control-flow.md)     | Function shape, abstraction levels, and readable control flow                | Writing or simplifying operations and branching                           |
| [references/code-quality/objects-and-responsibilities.md](references/code-quality/objects-and-responsibilities.md) | Cohesive ownership, encapsulation, and valid states                          | Designing classes/modules or fixing mixed responsibilities                |
| [references/code-quality/comments-and-formatting.md](references/code-quality/comments-and-formatting.md)           | When comments earn their keep; locality and vertical structure               | Commenting intent or organizing a file for reading                        |
| [references/code-quality/errors-and-resources.md](references/code-quality/errors-and-resources.md)                 | Failure meaning, cleanup, retries, and resource safety                       | Designing catch/return paths or reviewing failure handling                |
| [references/code-quality/boundaries-and-dependencies.md](references/code-quality/boundaries-and-dependencies.md)   | Trust boundaries, vendor isolation, and dependency direction                 | Integrating externals or reviewing I/O and coupling                       |
| [references/code-quality/concurrency.md](references/code-quality/concurrency.md)                                   | Shared state, ordering, cancellation, and concurrent correctness             | Writing or reviewing async/parallel code                                  |
| [references/code-quality/testing.md](references/code-quality/testing.md)                                           | Tests as evidence of observable behavior                                     | Adding, changing, or evaluating tests                                     |
| [references/code-quality/smells.md](references/code-quality/smells.md)                                             | Smell catalog as investigation prompts, not automatic verdicts               | Suspecting friction but needing a concrete lens                           |
| [references/refactoring/safe-refactoring.md](references/refactoring/safe-refactoring.md)                           | Definition, constraints, and evidence rules for behavior-preserving change   | Before any refactor; deciding whether and how to proceed                  |
| [references/refactoring/local-refactorings.md](references/refactoring/local-refactorings.md)                       | Extract/inline, temps, parameters, and small mechanical moves                | Improving a function or block without changing structure much             |
| [references/refactoring/structural-refactorings.md](references/refactoring/structural-refactorings.md)             | Move, extract/inline components, and encapsulate ownership                   | Relocating behavior or clarifying module boundaries                       |
| [references/refactoring/architectural-refactoring.md](references/refactoring/architectural-refactoring.md)         | Incremental, releasable shifts in layering and variation                     | Separating domain/presentation or evolving inheritance/composition        |
| [references/architecture/index.md](references/architecture/index.md)                                               | Entry point and review sequence for system design catalogs                   | Choosing which architecture topic to load next                            |
| [references/architecture/layering-and-domain-logic.md](references/architecture/layering-and-domain-logic.md)       | Layering, domain logic styles, and dependency direction                      | Organizing presentation, domain, and data concerns                        |
| [references/architecture/data-access.md](references/architecture/data-access.md)                                   | Gateways, Active Record, Data Mapper, and related access styles              | Choosing how application code talks to storage                            |
| [references/architecture/relational-mapping.md](references/architecture/relational-mapping.md)                     | Identity, mapping, and object↔relational translation                         | Loading/saving objects and avoiding duplicate identity bugs               |
| [references/architecture/relational-behavior.md](references/architecture/relational-behavior.md)                   | Unit of Work, Lazy Load, and transactional persistence behavior              | Multi-object writes, flush boundaries, or deferred loading                |
| [references/architecture/relational-structure.md](references/architecture/relational-structure.md)                 | Identity fields, foreign keys, inheritance, and value mapping                | Modeling relationships and row structure in the DB                        |
| [references/architecture/metadata-and-queries.md](references/architecture/metadata-and-queries.md)                 | Metadata mapping, query objects, and repository-style query APIs             | Reducing repetitive mapping or composing dynamic queries                  |
| [references/architecture/web-presentation.md](references/architecture/web-presentation.md)                         | Controllers, views, and request/response presentation patterns               | Designing or reviewing web/API request handling                           |
| [references/architecture/session-state.md](references/architecture/session-state.md)                               | Client vs server session placement, trust, and lifetime                      | Persisting state across requests safely                                   |
| [references/architecture/distribution.md](references/architecture/distribution.md)                                 | Remote boundaries, facades, and distributed-operation design                 | Crossing process/network boundaries or reviewing chatty remotes           |
| [references/architecture/concurrency-control.md](references/architecture/concurrency-control.md)                   | Optimistic/pessimistic offline locks and aggregate consistency               | Preventing lost updates across long-lived edits                           |
| [references/architecture/supporting-patterns.md](references/architecture/supporting-patterns.md)                   | Gateway, Mapper, Plugin, Service Layer, and related helpers                  | Isolating vendors, translating representations, or sharing layer behavior |
