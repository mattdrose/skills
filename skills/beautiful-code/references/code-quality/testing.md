# Testing

Use tests as fast, recent evidence that meaningful behavior still works. Treat untested behavior as an assumption and a passing suite as support for judgment, not proof of universal correctness.

## Test observable behavior

- State each scenario's setup, action, and expected outcome clearly.
- Exercise changed paths through stable public boundaries.
- Assert returned results, durable side effects, errors, and state transitions—not private helpers, call sequences, caches, or exact structure.
- Prefer high-value integration tests that survive implementation changes and fail on user-visible regressions.
- Use realistic inputs and small fixtures so relevant differences remain visible.
- Make failures identify the broken behavior without a debugging expedition.
- Reject weak assertions and setups that merely return what the test expects.
- Use tests to reveal coupling, ambiguous interfaces, and neglected edge cases.

## Match coverage to risk

- Cover representative successes, failures, boundaries, regressions, state transitions, ordering, persistence, and integration points.
- Broaden verification as the size and coupling of the change increase. Rerun enough connected behavior to catch effects beyond the edited unit.
- Test environment-dependent claims in representative environments.
- Add property-based tests for broad invariants such as round-trip equivalence, ordering, conservation, and idempotency. Preserve minimal failures as regression cases; inspect whether a failure exposes a defect, an incomplete property, or an unstated requirement.
- Characterize important legacy behavior as it exists, even when odd. Change that behavior separately.
- Supplement tests with type checking, static analysis, contract tests, generated-artifact snapshots, targeted manual checks, and production signals when pre-release environments cannot reproduce real data or conditions.
- State where evidence is weak and precisely qualify what was observed, where, and when.

## Control nondeterminism

- Control clocks, randomness, concurrency, and external state explicitly.
- Replace timing sleeps with a deterministic completion signal.
- Isolate or remove flaky and obsolete tests; never let them create false confidence.
- Do not weaken assertions, skip checks, or add retries merely to make the suite green. Fix the race, contract, setup, or product behavior.
- Make background failures observable rather than swallowing them.

## Prefer real boundaries over implementation mocks

- Prefer lightweight real implementations, in-memory stores, local servers, or boundary fakes over broad mocks that reproduce internals.
- Mock only a boundary that is impractical, unsafe, expensive, or nondeterministic to exercise directly.
- Keep the real contract represented when substituting a boundary; otherwise a passing test may confirm a fiction.
- Do not mock away the path whose behavior the test claims to verify.

## Refactoring safety

1. Identify observable contracts at the boundary being changed.
2. Run existing checks and start from a known-good state.
3. Add characterization coverage for important uncovered behavior, including failures and side effects.
4. Separate structural cleanup from behavior changes when possible.
5. Make one small, reversible structural change.
6. Run the narrowest relevant checks, then broader checks before finishing.
7. Revert when a failure cannot be explained by the last step.

Treat these as warning signs:

- Production behavior and assertions change together without explanation.
- Tests disappear because a new structure makes them inconvenient.
- Broad mocks encode implementation details.
- A large change is justified only by “the suite passes.”
- Flaky or skipped checks hide affected behavior.

## Examples

**Don't: pin internals and wait hopefully.**

```typescript
it("registers a user", async () => {
  const hashSpy = vi.spyOn(passwords, "hash");
  await service.register({ email: "ana@example.com", password: "s3cret!" });
  await new Promise((resolve) => setTimeout(resolve, 500));
  expect(hashSpy).toHaveBeenCalledOnce();
});
```

**Do: observe outcomes through the public boundary.**

```typescript
it("registers a user and sends a welcome email", async () => {
  const mailer = new FakeMailer();
  const service = new RegistrationService({ users: new InMemoryUsers(), mailer });

  const user = await service.register({
    email: "ana@example.com",
    password: "s3cret!",
  });

  expect(
    await service.authenticate({
      email: "ana@example.com",
      password: "s3cret!",
    }),
  ).toEqual(user);
  expect(mailer.sent).toContainEqual(expect.objectContaining({ to: "ana@example.com" }));
});
```

**Don't: let today's clock decide the result.**

```typescript
it("expires trials after 14 days", () => {
  const trial = new Trial(new Date("2026-07-03"));
  expect(trial.isExpired()).toBe(true);
});
```

**Do: supply time and test the boundary.**

```typescript
it("expires trials after 14 days", () => {
  const trial = new Trial(new Date("2026-07-03T00:00:00Z"));

  expect(trial.isExpiredAt(new Date("2026-07-16T23:59:59Z"))).toBe(false);
  expect(trial.isExpiredAt(new Date("2026-07-17T00:00:01Z"))).toBe(true);
});
```

**Don't: mock the answer under test.**

```typescript
it("returns the user's plan", async () => {
  vi.spyOn(planApi, "getPlan").mockResolvedValue("pro");
  expect(await getUserPlan("u1")).toBe("pro");
});
```

**Do: exercise the real storage boundary.**

```typescript
it("returns the stored plan", async () => {
  await db.seed({ users: [{ id: "u1", plan: "pro" }] });
  expect(await getUserPlan("u1")).toBe("pro");
});
```
