# Distribution

Minimize remote boundaries and design each one as a failure boundary.

## Interface

Expose coarse-grained operations through a remote facade. Transfer explicit data contracts rather than distributed domain objects.

## Failure

Specify timeouts, cancellation, retries, idempotency, partial failure, and compatibility. Do not disguise remote calls as local method calls.

## Placement

Distribute for an operational need—ownership, scaling, isolation, or geography—not solely for code organization.

## Review

Count network round trips, bound payloads, propagate tracing context, authenticate each boundary, and verify retry behavior cannot duplicate effects.

## Examples

### Don't: a remote service used like a local object

```typescript
const cart = await remoteCart.getCart(userId);
for (const item of cart.items) {
  // one network call per item, no timeout, retries can double-apply
  const price = await remotePricing.getPrice(item.sku);
  await remoteCart.updateItemPrice(userId, item.id, price);
}
```

### Do: one coarse, failure-aware operation

```typescript
const result = await pricingApi.repriceCart(
  { userId, skus: cart.items.map((i) => i.sku) },
  { timeoutMs: 2000, idempotencyKey: `reprice-${userId}-${cart.version}` },
);

if (result.kind === "timeout") return staleCartWithWarning(cart);
applyRepricedCart(result.cart);
```
