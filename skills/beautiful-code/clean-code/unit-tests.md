# Tests

Tests should provide fast evidence that externally meaningful behavior still works.

- Prefer integration-level assertions over tests coupled to private functions or call sequences.
- Make each scenario's setup, action, and expected outcome easy to identify.
- Cover important success, failure, boundary, and regression cases.
- Keep tests deterministic: control time, randomness, concurrency, and external state deliberately.
- Assert outcomes and durable side effects, not incidental implementation structure.
- Use realistic inputs; excessive mocking can confirm a fiction rather than the integration.
- Keep fixtures small enough that relevant differences are visible.
- A failing test should identify the broken behavior without requiring a debugging expedition.

Do not weaken assertions, add sleeps, or skip tests to make a suite green. Fix the race, contract,
or product behavior.

## Examples

### Don't: assert internals and synchronize with sleeps

```typescript
it("registers a user", async () => {
  const hashSpy = jest.spyOn(passwords, "hash");
  await service.register("ana@example.com", "s3cret!");
  await new Promise((resolve) => setTimeout(resolve, 500)); // hope the email went out
  expect(hashSpy).toHaveBeenCalledTimes(1); // pins the implementation, not the behavior
});
```

### Do: assert observable outcomes deterministically

```typescript
it("registers a user and sends a welcome email", async () => {
  const mailer = new FakeMailer();
  const service = new RegistrationService({
    users: new InMemoryUsers(),
    mailer,
  });

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

### Don't: let real time decide the outcome

```typescript
it("expires trials after 14 days", () => {
  const trial = new Trial(new Date("2026-07-03"));
  expect(trial.isExpired()).toBe(true); // passes or fails depending on the day the suite runs
});
```

### Do: control time deliberately

```typescript
it("expires trials after 14 days", () => {
  const trial = new Trial(new Date("2026-07-03T00:00:00Z"));

  expect(trial.isExpiredAt(new Date("2026-07-16T23:59:59Z"))).toBe(false);
  expect(trial.isExpiredAt(new Date("2026-07-17T00:00:01Z"))).toBe(true);
});
```
