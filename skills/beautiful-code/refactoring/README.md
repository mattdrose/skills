# Refactoring Review Guide

A compact reference for reviewing structural code changes. Use it to identify design pressure, choose the smallest safe transformation, and verify that behavior remains unchanged.

## Topic index

1. [Principles and timing](principles-in-refactoring.md) — purpose, constraints, and when to stop
2. [Code smells](bad-smells-in-code.md) — diagnostic signals and likely responses
3. [Safety and tests](building-tests.md) — establish observable behavior before changing structure
4. [Composing methods](composing-methods.md) — clarify control flow and local computation
5. [Moving responsibilities](moving-features-between-objects.md) — improve ownership and boundaries
6. [Organizing data](organizing-data.md) — make states, relationships, and invariants explicit
7. [Simplifying conditionals](simplifying-conditional-expressions.md) — expose decisions and remove branching noise
8. [Improving interfaces](making-method-calls-simpler.md) — make APIs intention-revealing and hard to misuse
9. [Reshaping inheritance](dealing-with-generalization.md) — simplify hierarchies and choose delegation deliberately
10. [Large-scale refactoring](big-refactorings.md) — evolve architecture incrementally
11. [Review workflow](putting-it-all-together.md) — a practical end-to-end checklist

## Core rule

Refactoring changes internal structure without intentionally changing observable behavior. Keep each step small, run relevant checks after each step, and separate cleanup from feature or bug-fix work. A reviewer should be able to explain both the design improvement and the evidence that behavior was preserved.
