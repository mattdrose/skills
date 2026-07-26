# Review Workflow

## Establish intent and risk

- State the user or operator problem, expected behavior, and explicit non-goals.
- Confirm that evidence supports the problem and that an existing capability does not already solve it.
- Identify trust boundaries, irreversible decisions, migrations, data loss risks, external systems, and operational impact.
- Trace every changed file to the requested outcome; challenge speculative flexibility and unrelated cleanup.
- Read surrounding code and local conventions before judging the patch.

## Verify behavior

- Follow the primary path end to end through public boundaries.
- Compare implementation behavior with requirements, including edge cases and compatibility expectations.
- Check input validation, state transitions, persistence, outputs, and side effects.
- Look for invalid states, stale assumptions, duplicated rules, off-by-one errors, precision loss, and partial writes.
- Reproduce suspected defects or derive a concrete failing case; do not infer failure from unfamiliar style.
- Confirm behavior outside the stated scope remains unchanged.

## Review clarity and control flow

- Read entry points first; require important decisions and control flow to remain discoverable.
- Check that names reveal domain purpose, units, representation, and side effects.
- Prefer direct branches and loops over clever compression, hidden callbacks, or unnecessary indirection.
- Flag comments that contradict code or replace clear structure; retain comments that explain constraints and tradeoffs.
- Delete dead code, pass-through layers, stale compatibility paths, and unused options when safe.
- Report unclear code only when it can cause misunderstanding, missed cases, or unsafe changes.

## Review responsibilities and dependencies

- Check that each component has a focused responsibility and a narrow interface.
- Keep policy separate from mechanism and domain concepts separate from vendor representations.
- Identify hidden dependencies, global state, internal-data chains, and unrelated concerns that must change together.
- Ensure each rule or fact has one authoritative source; do not demand reuse for merely similar syntax.
- Require boundaries around demonstrated volatility or external complexity, not imagined future variation.
- Check dependency quality, maintenance, interoperability, and whether the platform already provides the capability.

## Review failure, security, and concurrency

- Trace failure paths: rejection, timeout, cancellation, retries, partial completion, cleanup, and recovery.
- Preserve useful context in errors without exposing secrets, credentials, personal data, or internal details.
- Validate and normalize untrusted input at trust boundaries; parameterize queries and encode outputs for their context.
- Authenticate identity and authorize every protected action; default to denial and least authority.
- Keep secrets out of source, logs, errors, URLs, and client-visible payloads.
- Check retry safety, idempotency, transaction boundaries, resource release, and observability.
- For shared state, identify ownership and ordering; check races, deadlocks, lost updates, duplicate work, and stale reads.
- Do not accept a happy-path fix that can corrupt data or weaken security under failure.

## Review tests and change safety

- Require coverage of important observable behavior through stable public boundaries.
- Include the reported regression, credible edge cases, and consequential failure paths.
- Avoid assertions against private fields, call order, or replaceable implementation details.
- Check that tests can fail for the defect they claim to detect.
- Run relevant tests, static checks, and integration checks; record gaps instead of assuming coverage.
- Verify migrations, rollout compatibility, monitoring, rollback, and cleanup where the change requires them.
- Prefer a small production-shaped integration test over many isolated implementation tests.

## Report concrete findings

- Report defects and risks, not taste.
- Describe the triggering condition, observable consequence, and affected user, data, or operation.
- Point to the smallest relevant location and distinguish confirmed failures from unresolved questions.
- Rank findings by impact and likelihood; lead with correctness, security, data loss, and outages.
- Recommend the smallest fix that removes the consequence without broadening scope.
- Do not request renaming, abstraction, formatting, or refactoring unless it prevents a concrete misunderstanding or unsafe change.
- State what was verified and what remains unverified.

Use this form:

> **Finding:** Concurrent retries can charge the same order twice. Both requests pass the status check before either writes `paid`; the payment call has no idempotency key. Send `order.id` as the provider idempotency key and make the status transition atomic.
