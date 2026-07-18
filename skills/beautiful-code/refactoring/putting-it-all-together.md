# Refactoring Review Workflow

## Before

- State the design problem and the behavior that must remain unchanged.
- Confirm that refactoring is needed for current work.
- Identify public contracts, side effects, persistence, concurrency, and performance constraints.
- Begin from passing relevant checks; add characterization coverage where risk demands it.

## During

- Make one explainable transformation at a time.
- Keep behavior changes separate.
- Run focused checks after each meaningful step.
- Prefer deletion, direct code, and existing language features over new abstractions.
- Record unrelated discoveries instead of expanding scope.
- Stop and revert to the last known-good state when a failure cannot be tied to the latest step.

## Review the result

- **Intent:** Is the maintenance benefit concrete?
- **Behavior:** What evidence shows contracts were preserved?
- **Ownership:** Are rules now closer to the data or lifecycle they govern?
- **Readability:** Do names and boundaries expose intent?
- **Dependencies:** Is coupling reduced rather than merely relocated?
- **API:** Are compatibility and error semantics preserved?
- **Data:** Are invariants and migrations safe?
- **Scope:** Is every changed line necessary for this refactoring?
- **Residue:** Are old paths, temporary adapters, comments, dead imports, and duplicate tests removed?

## Finish

Run the full applicable test, type, lint, and build checks. Inspect the diff from the perspective of someone unfamiliar with the change. Document only enduring constraints or non-obvious decisions. If behavior also needs to change, make that a distinct follow-up whose effect can be reviewed independently.

## Examples

### Don't: refactor without a workflow

A developer decides the order module is "messy," spends a week restructuring it on a long-lived
branch without tests, and folds in a pricing change discovered along the way. The final diff touches
forty files, cannot be verified step by step, and is rejected in review.

### Do: work the checklist

The same developer needs to add a new shipping option and finds the order module hard to extend.
They state that goal, add characterization tests around order submission, then land three small
reviewed steps: extract shipping selection, move it behind an interface, delete the old branching.
The shipping feature follows as a separate, easily reviewed change.
