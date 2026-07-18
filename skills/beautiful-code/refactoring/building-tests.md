# Safety and Tests

Tests provide feedback that structural changes preserve behavior. Refactoring without useful feedback is guesswork.

## Establish a safety net

1. Identify observable contracts at the boundary being changed.
2. Run the existing suite and begin from a known-good state.
3. Add characterization tests for important uncovered behavior, including errors and side effects.
4. Make one structural change.
5. Run the narrowest relevant checks, then broader checks before finishing.
6. Revert immediately when a failure cannot be explained by the last step.

Prefer tests through stable public boundaries. Avoid assertions about private helpers, call sequences, or exact structure: those tests obstruct the refactoring they should enable.

### Don't: assert on internal structure and call sequences

```typescript
test("applies discount", () => {
  const calc = new PriceCalculator();
  const spy = jest.spyOn(calc as any, "lookupDiscountRate"); // private helper
  calc.priceFor(order);
  expect(spy).toHaveBeenCalledWith("gold"); // breaks when the helper is renamed
  expect((calc as any).cachedRate).toBe(0.1); // breaks when the cache is removed
});
```

### Do: test the observable contract

```typescript
test("gold customers get 10% off", () => {
  const calc = new PriceCalculator();
  const order = makeOrder({ customerTier: "gold", subtotal: 200 });
  expect(calc.priceFor(order)).toBe(180);
});
```

## Choose coverage by risk

Cover representative success paths, boundary values, failures, state transitions, ordering, persistence, and integration points. For legacy behavior, characterize what the system does now—even when odd—then change that behavior separately if required.

Tests are not the only evidence. Type checking, static analysis, contract tests, snapshots of generated artifacts, and targeted manual checks can supplement them. State explicitly when confidence depends on such checks or when coverage remains weak.

## Review red flags

- production behavior and assertions change together without explanation
- tests are deleted because the new structure makes them inconvenient
- broad mocks reproduce implementation details
- a large change is justified only by “the suite passes”
- flaky or skipped checks hide the affected behavior
