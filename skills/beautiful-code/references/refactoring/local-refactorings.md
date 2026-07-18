# Local Refactorings

## Extract and compose operations

- **Extract function:** Trigger: a block expresses a nameable rule, repeats, or mixes abstraction levels. Improvement: the top-level operation reads as a workflow and the rule gains a focused home. Caution: account for inputs, outputs, mutation, errors, and timing; do not scatter clear code into tiny weakly named helpers.
- **Inline function:** Trigger: indirection communicates no more than its body. Improvement: restores a direct, readable flow. Caution: retain a boundary that carries domain meaning, reuse, substitution, or isolation.
- **Introduce a function object:** Trigger: a genuinely complex computation needs shared intermediate state. Improvement: groups that state and its steps coherently. Caution: it is heavier than extraction; do not create an object merely to shorten a function.
- **Substitute algorithm:** Trigger: equivalent mechanics are convoluted or costly for the evidenced workload. Improvement: a clearer adequate algorithm exposes intent. Caution: preserve edge cases, ordering, errors, side effects, and complexity guarantees; measure before optimizing.

## Clarify temporary values

- **Replace temporary with query:** Trigger: a derived value obscures its meaning or repeats. Improvement: names the derivation and can remove local state. Caution: repeated computation must remain safe, sufficiently cheap, and free of newly hidden side effects.
- **Split variable:** Trigger: one variable represents different meanings over its lifetime. Improvement: each value gets a stable, intention-revealing name. Caution: preserve assignment order and mutation semantics.
- **Avoid parameter assignment:** Trigger: an input is overwritten and loses its original meaning. Improvement: a local value makes input and derived state distinct. Caution: preserve aliasing and caller-visible mutation where the language permits it.

## Simplify conditionals

- **Extract condition or branch:** Trigger: a decision or action is non-obvious. Improvement: a domain name reveals policy. Caution: preserve evaluation order, short-circuiting, exceptions, and side effects.
- **Consolidate conditions:** Trigger: several checks implement one rule and lead to the same outcome. Improvement: removes branching noise and exposes the shared policy. Caution: do not merge policies that merely coincide today but may evolve independently.
- **Deduplicate branches:** Trigger: branches repeat identical setup or cleanup. Improvement: leaves only genuine variation in the decision. Caution: moving code must not alter timing, partial mutation, or exception cleanup.
- **Use guard clauses:** Trigger: terminal or exceptional cases bury the normal path. Improvement: precedence and main flow become visible. Caution: do not bypass required cleanup or conceal partially completed work.
- **Remove control flags:** Trigger: a mutable flag simulates return, break, or extracted flow. Improvement: direct control flow is shorter and clearer. Caution: retain required traversal, cleanup, and first-versus-last-match semantics.
- **Replace repeated type branching:** Trigger: the same stable variant switch appears across callers. Improvement: variant behavior gains one owner through a strategy or polymorphism. Caution: a small local switch may be simpler; preserve exhaustiveness and do not build a hierarchy for speculative variants.
- **Model absence explicitly:** Trigger: null handling is repeated and absence has uniform domain meaning. Improvement: an option, result, or null object centralizes semantics. Caution: do not hide meaningful distinctions, failures, or expensive behavior behind a surprising default.
- **Assert internal invariants:** Trigger: an impossible state indicates a programmer defect. Improvement: failures occur near the violated assumption. Caution: assertions do not replace validation of untrusted input or ordinary error handling.

## Separate queries from commands

Trigger: an operation presented as a question unexpectedly mutates state. Expected improvement: callers can reason separately about observation and mutation, and dependencies become visible. Principal caution: preserve required atomicity, ordering, audit behavior, and compatibility; splitting a transaction can create races.

## Replace selector flags with explicit operations

Trigger: a boolean or mode parameter selects distinct caller intentions. Expected improvement: named entry points make call sites self-explanatory and prevent invalid modes. Principal caution: avoid a proliferation of operations when the selector is genuine data; preserve defaults and migrate public callers incrementally.

## Improve parameter and return contracts

- **Rename an operation:** Trigger: its name no longer matches domain language or observable effect. Improvement: callers understand intent without reading the body. Caution: public names require compatibility migration.
- **Remove or add a parameter:** Trigger: an input is unused, or a caller legitimately owns a choice now hidden internally. Improvement: the signature reflects responsibility. Caution: preserve evaluation order, defaults, testability, and binary/source compatibility.
- **Preserve a whole object:** Trigger: callers repeatedly unpack several fields belonging to one domain concept. Improvement: fewer arguments and preserved context. Caution: do not couple the callee to a broad object it should not know.
- **Introduce a parameter object:** Trigger: a recurring parameter group has shared meaning or validation. Improvement: invalid combinations can be constrained and signatures become stable. Caution: a bag of unrelated options only hides complexity.
- **Replace parameter with query:** Trigger: the receiver properly owns and can obtain the dependency. Improvement: removes redundant caller knowledge. Caution: do not hide I/O, expensive work, global state, or an untestable dependency.
- **Narrow the supported interface:** Trigger: setters or operations expose unsupported mutation. Improvement: invariants gain one enforcement point. Caution: identify real consumers before removal and provide a migration path.
- **Use a factory:** Trigger: construction requires validation, policy, caching, or subtype selection. Improvement: construction intent and invariants become explicit. Caution: do not conceal expensive I/O or create ceremony around trivial construction.
- **Improve failure contracts:** Trigger: error codes are ignored or exceptions are used for expected branching. Improvement: typed failures, exceptions, or precondition checks fit the surrounding contract. Caution: preserve failure categories, messages where contractual, and transaction behavior; validate at trust boundaries.

## Compatibility and migration

For a public interface, add the new entry point, migrate callers in small groups, deprecate the old one, and remove it only when compatibility permits. Preserve errors, defaults, nullability, ordering, side effects, and evaluation timing unless behavior change is separately requested. Temporary adapters should be bounded and observable, not permanent duplicate APIs.
