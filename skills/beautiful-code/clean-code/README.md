# Clean Code Review Guide

A concise, language-neutral reference for reviewing code clarity and maintainability. Treat these as
prompts, not laws: preserve working behavior, follow local conventions, and prefer evidence over
taste.

## Review path

1. [Core principles](clean-code.md)
2. [Names](meaningful-names.md)
3. [Functions](functions.md)
4. [Comments](comments.md)
5. [Formatting](formatting.md)
6. [Objects and data](objects-and-data-structures.md)
7. [Error handling](error-handling.md)
8. [Boundaries](boundaries.md)
9. [Tests](unit-tests.md)
10. [Classes and modules](classes.md)
11. [System design](systems.md)
12. [Concurrency](concurrency.md)
13. [Smells checklist](smells-and-heuristics.md)

## How to use this collection

Start with correctness and risk. Then review names, control flow, responsibilities, dependencies,
and tests. Report concrete consequences—misleading behavior, unsafe change paths, hidden
coupling—not mere stylistic disagreement. Recommend the smallest change that resolves the problem.
