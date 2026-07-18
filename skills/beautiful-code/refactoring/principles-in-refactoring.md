# Principles and Timing

Refactoring improves the internal design of working software while preserving its observable behavior. It is not a feature change, bug fix, or speculative rewrite.

## Review principles

- **One hat at a time:** distinguish structural changes from behavior changes. Commit or review them separately when practical.
- **Small verified steps:** prefer transformations that are easy to understand, test, and reverse.
- **Design for the next change:** improve the code needed for current work rather than pursuing an imagined ideal architecture.
- **Clarity over cleverness:** names, boundaries, and explicit control flow matter more than reducing line count.
- **Preserve contracts:** outputs, errors, side effects, ordering, timing guarantees, and public types may all be observable.
- **Measure performance:** structural clarity comes first unless profiling identifies a real constraint.

### Don't: mix hats in one change

While fixing a rounding bug in invoice tax calculation, the author also renames three classes,
reorders parameters on a public API, and rewrites the discount logic "while in there." The reviewer
cannot tell which lines changed behavior, and the rounding fix ships two days late.

### Do: separate structure from behavior

First commit: extract the tax calculation into a named function and add characterization tests, with
output unchanged. Second commit: fix the rounding bug inside that function, with one test updated to
assert the corrected value. Each commit is reviewable and reversible on its own.

## When to refactor

Refactor while preparing to add behavior, after discovering duplicated knowledge, or when a defect reveals a confusing boundary. Repeated friction is stronger evidence than aesthetic preference.

Avoid broad cleanup during an urgent fix, near an unstable release, or where behavior cannot be checked. First add characterization coverage or reduce the scope.

## Stopping rule

Stop when the intended change becomes straightforward, the relevant smell is removed, or the next step cannot be justified confidently. Leave unrelated opportunities for later. A partial improvement that remains coherent is better than an uncontrolled redesign.
