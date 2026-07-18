# Small, Evidence-Based Changes

Every changed line can introduce a defect. Keep the change no larger than the verified problem requires.

## Review prompts

- Is the problem reproduced, measured, or supported by user evidence?
- Does each changed file contribute to the requested behavior?
- Can the work be split into independently useful, verifiable steps?
- Is unrelated cleanup increasing review or rollback risk?
- Does the patch preserve existing behavior outside its stated scope?
- Is performance work aimed at a measured bottleneck?

Small does not mean incomplete. A safe change still includes necessary validation, migration, error handling, observability, documentation, and tests.

When a feature is difficult to add, improve only the design needed to make that feature straightforward, then implement it. Avoid broad rewrites unless evidence shows incremental repair is less safe or more costly and both old and new systems can be supported through the transition.

### Don't: bundle a one-line fix with speculative rework

```typescript
// The reported bug is tax rounding, but this patch also adds caching and
// configuration for load that was never measured.
const rateCache = new Map<string, { rate: number; fetchedAt: number }>();

async function calculateTax(order: Order, opts: { cacheTtlMs?: number } = {}): Promise<number> {
  const ttl = opts.cacheTtlMs ?? 60_000;
  const cached = rateCache.get(order.region);
  const rate =
    cached && Date.now() - cached.fetchedAt < ttl ? cached.rate : await fetchRate(order.region);
  rateCache.set(order.region, { rate, fetchedAt: Date.now() });
  return Math.round(order.subtotal * rate * 100) / 100;
}
```

### Do: fix exactly the verified defect

```typescript
async function calculateTax(order: Order): Promise<number> {
  const rate = await fetchRate(order.region);
  // Round in cents to avoid the floating-point drift reported in #4821.
  return Math.round(order.subtotal * rate * 100) / 100;
}
```
