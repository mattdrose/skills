# Structural Refactorings

## Move behavior toward its owner

- **Move function or field:** Trigger: behavior lives away from the data, invariant, or lifecycle it primarily serves. Improvement: reduces navigation, leakage, and coordinated edits. Caution: preserve mutation, transaction, visibility, and initialization semantics; do not merely hide coupling or create a cycle.
- **Add a local adapter:** Trigger: an external type lacks behavior that cannot safely be added upstream. Improvement: isolates foreign concepts behind a local contract. Caution: prevent the adapter from leaking upstream details or becoming a second source of truth.

Choose the owner that holds the invariant, contains most required data, changes for the same reason, and can expose the narrowest stable contract—not whichever move removes an import.

## Extract cohesive responsibilities

- **Extract component:** Trigger: one unit contains cohesive responsibilities with independent reasons to change. Improvement: gives each responsibility focused ownership and lifecycle. Caution: avoid chatty boundaries, broadened visibility, duplicate state, or splitting behavior that must remain atomic.
- **Inline component:** Trigger: a type adds pass-through indirection without useful meaning or isolation. Improvement: simplifies navigation and removes a false boundary. Caution: retain boundaries that protect invariants, external dependencies, or independent change.

## Encapsulate delegation and collections

- **Hide delegate:** Trigger: callers traverse internal object graphs to perform an intention-level operation. Improvement: reduces caller knowledge and stabilizes navigation behind the owner. Caution: do not grow a broad pass-through API or conceal meaningful collaborator ownership.
- **Remove middleman:** Trigger: a layer does nothing but delegate and obscures the real collaborator. Improvement: removes needless indirection. Caution: ensure the middleman is not enforcing policy, compatibility, authorization, or lifecycle rules.
- **Encapsulate mutable fields and collections:** Trigger: aliases can mutate state while bypassing invariants. Improvement: guarded operations establish one mutation boundary and readonly views protect internals. Caution: preserve ordering, identity, iteration, performance, and serialization behavior; a readonly type alone may not prevent runtime mutation.
- **Choose one relationship authority:** Trigger: bidirectional links or cached derived values drift. Improvement: one source of truth makes updates and derivation consistent. Caution: update required counterparts atomically and account for persistence and concurrency.

## Replace primitives with domain values

- **Replace related primitives or positional data:** Trigger: units, validation, or field meaning are ambiguous and invalid combinations are representable. Improvement: named values centralize invariants and equality. Caution: a wrapper that neither constrains nor clarifies adds ceremony; validate external input before construction.
- **Name a magic literal:** Trigger: a repeated literal carries domain meaning. Improvement: names policy and prevents accidental inconsistency. Caution: do not centralize coincidentally equal values that may vary independently.
- **Distinguish entities from values:** Trigger: equality or mutation semantics are unclear. Improvement: identity-bearing entities and immutable values receive appropriate contracts. Caution: preserve persistence identity, hashing, and collection behavior.
- **Model lifecycle states explicitly:** Trigger: nullable or temporary fields permit impossible state combinations. Improvement: valid transitions and absence semantics become explicit. Caution: include legacy and failure states required by real data; do not silently reject persisted cases.

## Replace type codes with explicit variants

Trigger: a raw code admits invalid values or repeated branching, and variants have stable, genuinely different behavior. Expected improvement: a constrained type prevents invalid codes; explicit variants, subclasses, or strategies give behavior a single owner. Principal caution: use a simple union when only validation varies, keep a small local switch when clearer, and preserve exhaustive handling, serialization, and unknown-value behavior.

## Reshape inheritance

- **Pull members up:** Trigger: demonstrated subtypes share behavior or state with identical meaning. Improvement: removes true duplication and centralizes a common contract. Caution: structurally similar members may represent different policies; avoid speculative base classes.
- **Push members down:** Trigger: only particular variants support a parent member. Improvement: narrows the parent contract and prevents unsupported operations. Caution: migrate callers relying on the broad parent and preserve substitutability.
- **Extract a subtype:** Trigger: a coherent subset has distinct behavior callers benefit from recognizing. Improvement: makes variation and valid operations explicit. Caution: do not encode a temporary flag as a permanent hierarchy.
- **Extract a superclass or interface:** Trigger: multiple types already honor a meaningful common contract. Improvement: enables shared use without duplicating that contract. Caution: marker interfaces and reuse-only parents create false substitutability.
- **Collapse hierarchy:** Trigger: variation has disappeared or levels add no meaning. Improvement: removes navigation and override complexity. Caution: preserve construction, dispatch, public types, and extension points still in use.
- **Form template method:** Trigger: an algorithm is stable while a few explicit steps vary. Improvement: centralizes sequencing and keeps variation focused. Caution: inheritance couples variants to the algorithm; composition may fit independently changing steps better.

## Prefer delegation when substitution does not hold

- **Replace inheritance with delegation:** Trigger: a subtype reuses implementation but cannot honor the complete parent contract or protect its invariants. Improvement: exposes only supported operations and makes collaboration explicit. Caution: preserve needed behavior without recreating the entire parent interface as pass-through methods.
- **Replace delegation with inheritance:** Trigger: the delegate is genuinely substitutable and its complete interface should be exposed. Improvement: can remove needless forwarding. Caution: this is rare; verify every inherited mutation, error, and side-effect expectation, not just shared shape.

Ask whether every subtype can replace its parent without surprises, whether overrides preserve invariants and errors, and whether variation is independent, optional, or combinable. Prefer composition in those latter cases; tolerate a little duplication rather than extracting the wrong relationship.

## Migration discipline

Introduce the new owner or representation at one boundary, migrate callers and stored data in small groups, and use a temporary delegate or adapter only for a bounded compatibility period. Preserve public contracts, equality, ordering, serialization and wire formats, database constraints, and transaction boundaries unless separately changing behavior. Monitor for cycles, aliases, dual writes, and duplicate sources of truth, then remove the old path promptly when migration permits.
