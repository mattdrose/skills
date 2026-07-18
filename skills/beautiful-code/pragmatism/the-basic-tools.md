# Working Tools

## Plain text

**Review question:** Are durable data and configuration inspectable with ordinary tools?

Prefer documented, versionable text formats where practical. They survive tool changes and support search, diff, automation, and recovery. Use binary formats when their concrete benefits outweigh that interoperability.

## Shell fluency

**Review question:** Can recurring repository tasks be composed and automated from the command line?

Learn the shell and core tools of the working environment. Preserve useful commands in scripts or task definitions so the result is repeatable rather than dependent on personal history.

## Editor fluency

**Review question:** Are routine editing operations fast, consistent, and available without manual repetition?

Know one editor deeply enough to navigate, transform, search, and automate text without friction. Use the same core setup across relevant languages and keep its configuration reproducible.

## Version control

**Review question:** Is every meaningful artifact versioned with a history that explains change?

Version source, tests, documentation, configuration, and operational scripts. Keep changes coherent and reviewable. Practice restoration and branching workflows before an emergency requires them.

## Debugging

**Review question:** Was the root cause demonstrated rather than guessed?

Reproduce first, gather evidence, and narrow the failing boundary systematically. Read the actual error, inspect recent changes, and challenge assumptions. Fix the cause, add protection against recurrence, and verify that the explanation fits every observed symptom.

### Don't: guess at the cause and patch the symptom

A request intermittently returns an empty cart, so the engineer adds a retry around the fetch and
closes the ticket without reproducing the failure; the underlying stale-cache bug keeps surfacing
in other endpoints.

### Do: reproduce, narrow, and demonstrate the root cause

The engineer writes a failing test that reproduces the empty cart, bisects to the commit that
changed cache invalidation, fixes the invalidation key, keeps the test as protection, and confirms
the fix explains every reported symptom.

## Text manipulation

**Review question:** Is repetitive transformation being delegated to a reliable tool?

Use search, structured queries, scripts, or one-off programs instead of manual editing. Make broad transformations previewable and verify them with version-control diffs and relevant tests.

## Engineering journal

**Review question:** Are decisions, experiments, and unexplained observations recorded where they can be recovered?

Keep a dated working record. It need not be polished: capture commands, results, diagrams, decisions, and open questions. Move durable team knowledge into shared documentation rather than leaving it only in private notes.
