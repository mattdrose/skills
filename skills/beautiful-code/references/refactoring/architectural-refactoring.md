# Architectural Refactoring

Architectural refactoring is a direction delivered through many small, releasable, behavior-preserving changes—not a broad rewrite or a long-lived branch.

## Separate domain and presentation

Trigger: business rules are embedded in UI, transport, formatting, or framework code. Expected improvement: domain policy can evolve and be verified independently while presentation translates through an explicit boundary. Principal caution: do not create an anemic pass-through layer or duplicate validation and calculations on both sides; preserve rendering, errors, and transaction semantics while moving one rule at a time.

A related move from procedures to domain ownership is warranted when procedures repeatedly manipulate the same records. Move one coherent rule to the unit owning its data and invariant, then migrate its callers; avoid designing a complete domain model upfront.

## Evolve inheritance into composition

Trigger: a hierarchy encodes reuse rather than substitutability, variants cannot honor the parent contract, or behavior is optional and combinable. Expected improvement: a narrow collaborator exposes supported behavior and allows variation without inheriting unrelated state or operations. Principal caution: delegation can become a verbose mirror of the old parent; migrate one behavior at a time and preserve dispatch, lifecycle, and compatibility until callers move.

## Separate independent dimensions of change

Trigger: one hierarchy or overloaded type combines axes that vary independently—such as report kind and output format—or fields and behavior have distinct lifecycles. Expected improvement: retain one coherent dimension and compose or extract the other, preventing a multiplying set of variants and coordinated edits. Principal caution: prove the axes are independent before adding interfaces; avoid duplicate state, chatty boundaries, and speculative combinations.

## Introduce boundaries incrementally

Trigger: tangled ownership blocks a concrete capability or repeatedly forces unrelated modules to change together. Expected improvement: a seam, boundary model, collaborator, or variant confines knowledge and lets one path migrate at a time. Principal caution: adapters are transitional tools, not a second architecture; bound their lifetime and watch for leaky contracts, cycles, dual writes, and inconsistent old and new paths.

Define the target direction without designing every intermediate detail. Keep persistence, protocols, and public contracts stable unless their change is explicitly separated, and retain validation at trust boundaries.

## Execution strategy

1. State the concrete capability or maintenance problem and the observable behavior that must remain unchanged.
2. Map seams, dependencies, ownership, data flow, persistence, and runtime constraints.
3. Establish characterization and boundary evidence; use [Testing](../code-quality/testing.md) for detailed guidance.
4. Describe the target direction and the next releasable step, not a complete replacement system.
5. Migrate one rule, caller group, or runtime path at a time; use a temporary adapter only when compatibility requires it.
6. Keep old and new paths consistent for a bounded transition, with a rollback or containment strategy for critical paths.
7. Deliver and reassess whether the next step remains justified.
8. Remove obsolete code, dual writes, and compatibility layers promptly when migration permits.

Do not fold feature changes into structural steps. Broad rewrites are justified only by evidence that incremental repair is less safe or more costly and both systems can be supported through transition.

## Review risks

Review for dual sources of truth, hidden data migrations, cycles, leaky adapters, broadened interfaces, multiplied variants, and branches that cannot ship or roll back independently. Verify that each boundary clarifies ownership rather than relocating coupling, and that error, ordering, concurrency, persistence, and protocol contracts remain observable.

Require containment for persistence, protocols, and critical runtime paths. Reject architecture pursued only for diagram conformity: success is easier product change, clearer ownership, and less coordinated editing. Stop when the driving capability becomes straightforward or the next migration step lacks evidence.
