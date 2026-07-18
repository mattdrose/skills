# Safe Refactoring

## Definition and constraints

Refactoring changes internal structure without intentionally changing observable behavior. Preserve outputs, errors, side effects, ordering, public types, persistence and wire formats, concurrency guarantees, and relevant performance characteristics. A feature change, bug fix, or speculative rewrite is not refactoring; separate it so each effect remains explainable and reversible.

Refactor toward the next evidenced need, not an imagined ideal. Prefer direct code, deletion, and existing language features over new machinery. Small does not mean incomplete: retain necessary validation, migration, observability, documentation, and checks. During structural change, preserve existing security behavior: least-authority and default-deny checks, input validation, encryption and confidentiality controls, dependency-integrity and security constraints, attack-surface boundaries, observability, and failure reporting that does not expose secrets. Dependency upgrades and other behavior-changing maintenance are separate work, not refactoring.

## When to refactor

Refactor when preparing a current behavior change, duplicated knowledge causes coordinated edits, a defect exposes a confused boundary, or repeated maintenance friction identifies a concrete design problem. The expected improvement should be specific: make the pending change straightforward, clarify ownership, or remove the demonstrated duplication.

Do not start broad cleanup from aesthetic discomfort alone, during an urgent fix, near an unstable release, or where behavior cannot yet be observed. Intuition is a trigger to gather evidence, not evidence itself. Estimate time and space growth against representative and worst credible inputs; optimize only measured bottlenecks.

## Establish observable behavior

State the design problem and enumerate the contracts that must not change, including failures and side effects. Begin from known-good checks and characterize important uncovered legacy behavior as it exists—even when odd. Detailed guidance on stable test boundaries, risk-based coverage, nondeterminism, and supplementary checks lives in [Testing](../code-quality/testing.md).

Testing supports judgment rather than proving correctness. Record weak coverage and constraints that cannot be reproduced; do not justify a large change only with “the suite passes.”

## Work in small steps

Make one explainable, reversible transformation at a time. Run the narrowest relevant checks after each meaningful step and broader checks before finishing. If a failure cannot be explained by the latest step, stop and revert to the last known-good state.

Keep each step releasable where practical. Migrate callers or data in bounded groups, use temporary adapters only when compatibility requires them, and remove old paths once migration permits. Record unrelated discoveries instead of expanding scope. Avoid long-lived rewrite branches: incremental repair is safer unless evidence shows otherwise and both paths can be supported during transition.

## Separate structural and behavioral changes

Wear one hat at a time. First reshape working code while preserving behavior; then make the feature or defect correction as a distinct change. When practical, review or commit those changes separately. Do not change production behavior and its assertions together without explaining exactly which contract intentionally changed.

This separation narrows diagnosis and rollback. Its caution is procedural overhead: do not manufacture meaningless steps, but never conceal behavioral change inside cleanup.

## Review the result

Review the diff as an unfamiliar maintainer:

- Is the maintenance benefit concrete, and is every changed line necessary?
- What evidence preserves each observable contract?
- Are rules closer to the data or lifecycle they govern, with less knowledge between modules?
- Do names, control flow, dependencies, side effects, and boundaries reveal intent?
- Are API compatibility, errors, invariants, migrations, concurrency, and measured performance preserved?
- Did coupling move rather than decrease, or were cycles and duplicate sources of truth introduced?
- Are temporary adapters, old paths, comments, imports, and duplicate checks removed when safe?

Run applicable test, type, lint, and build checks. Document only enduring constraints or non-obvious decisions.

## Stopping rule

Stop when the intended change is straightforward, the evidenced smell is removed, or the next step cannot be justified confidently. Reassess after every useful delivery. A coherent partial improvement is better than an uncontrolled redesign; leave unrelated opportunities for later.
