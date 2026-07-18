# Testing Behavior

Confidence comes from recent, accurate observation of software in the conditions that matter. Untested behavior is an assumption.

## Review prompts

- Does each test state a meaningful behavior and expected outcome?
- Does the suite exercise the changed path through its real boundary?
- Are important failures, edge cases, and state transitions covered?
- Will a failure be visible and diagnostic?
- Are environment-dependent claims tested in representative environments?
- Could a passing test conceal broken behavior through weak assertions or unrealistic setup?
- Are flaky or obsolete tests giving false confidence?

Prefer tests that survive implementation changes and fail when user-visible behavior regresses. After a change, rerun enough connected behavior to detect effects beyond the edited unit. The broader the change and coupling, the broader the required verification.

Tests establish what was observed, where, and when; they do not prove universal correctness. Keep their claims precise and supplement them with production signals when behavior depends on data or environments that cannot be reproduced fully before release.

### Don't: assert weakly on implementation details

```typescript
// Passes even if the discount is wrong; breaks if the internal method is renamed.
it("applies discount", () => {
  const spy = vi.spyOn(cart, "recalculate");
  cart.applyCoupon("SAVE20");
  expect(spy).toHaveBeenCalled();
  expect(cart.total).toBeDefined();
});
```

### Do: state the behavior and assert the outcome

```typescript
it("SAVE20 reduces a $100 cart to $80", () => {
  const cart = cartWithItems([{ price: 100 }]);
  cart.applyCoupon("SAVE20");
  expect(cart.total).toBe(80);
});

it("rejects an expired coupon and leaves the total unchanged", () => {
  const cart = cartWithItems([{ price: 100 }]);
  expect(() => cart.applyCoupon("EXPIRED")).toThrow(CouponExpiredError);
  expect(cart.total).toBe(100);
});
```

### Don't: mock away the behavior under test

```typescript
// The mock returns exactly what the assertion checks; no real path is exercised.
it("returns the user's plan", async () => {
  vi.spyOn(planApi, "getPlan").mockResolvedValue("pro");
  const plan = await getUserPlan("u1");
  expect(plan).toBe("pro");
});
```

### Do: exercise the changed path through its real boundary

```typescript
it("returns the plan stored for the user", async () => {
  await db.seed({ users: [{ id: "u1", plan: "pro" }] });
  const plan = await getUserPlan("u1");
  expect(plan).toBe("pro");
});
```
