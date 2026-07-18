# Smells Checklist

Smells are investigation prompts, not automatic findings.

## Intent

- misleading or inconsistent names
- magic values or unexplained units
- dense expressions and mixed abstraction levels
- comments compensating for unclear code

### Don't: hide intent behind magic values

```typescript
if (user.status === 3 && Date.now() - user.createdAtMs > 2592000000) {
  applyDiscount(user, 0.15);
}
```

### Do: name the values and the rule

```typescript
const LOYALTY_DISCOUNT_RATE = 0.15;
const THIRTY_DAYS_MS = 30 * 24 * 60 * 60 * 1000;

const isEstablishedMember =
  user.status === UserStatus.Active && Date.now() - user.createdAtMs > THIRTY_DAYS_MS;

if (isEstablishedMember) {
  applyDiscount(user, LOYALTY_DISCOUNT_RATE);
}
```

## Responsibility

- functions with unrelated phases
- classes that change for many reasons
- feature envy or behavior far from the data it governs
- low-level modules owning high-level policy

### Don't: navigate deep object graphs from afar

```typescript
// Caller depends on the entire graph's shape, and the tax rule lives far from its data
if (order.customer.profile.address.country === "CA") {
  taxCents = Math.round(order.subtotalCents * gstRateFor(order.customer.profile.address.province));
}
```

### Do: put the behavior with the data it governs

```typescript
class Order {
  taxCents(): number {
    return Math.round(this.subtotalCents * this.shippingAddress.taxRate());
  }
}

const taxCents = order.taxCents();
```

### Don't: repeat the same type switch across functions

```typescript
function shippingCost(parcel: Parcel): number {
  switch (parcel.kind) {
    case "letter":
      return 120;
    case "box":
      return 899;
    case "pallet":
      return 4500;
  }
}

function maxWeightGrams(parcel: Parcel): number {
  switch (
    parcel.kind // the same dispatch, duplicated; new kinds require finding every switch
  ) {
    case "letter":
      return 100;
    case "box":
      return 20_000;
    case "pallet":
      return 500_000;
  }
}
```

### Do: dispatch once on the variation

```typescript
interface ParcelSpec {
  shippingCostCents: number;
  maxWeightGrams: number;
}

const PARCEL_SPECS: Record<ParcelKind, ParcelSpec> = {
  letter: { shippingCostCents: 120, maxWeightGrams: 100 },
  box: { shippingCostCents: 899, maxWeightGrams: 20_000 },
  pallet: { shippingCostCents: 4500, maxWeightGrams: 500_000 },
};

const spec = PARCEL_SPECS[parcel.kind];
```

## Coupling

- long object-navigation chains
- hidden globals or service location
- dependency cycles
- broad interfaces and vendor types leaking inward
- base modules aware of concrete implementations

### Don't: fetch dependencies through hidden globals

```typescript
function chargeSubscription(sub: Subscription): Promise<Receipt> {
  const gateway = ServiceLocator.get("payments"); // invisible dependency, untestable wiring
  const logger = globalThis.appLogger;
  logger.info(`charging ${sub.id}`);
  return gateway.charge(sub.priceCents);
}
```

### Do: accept dependencies explicitly

```typescript
function chargeSubscription({
  sub,
  gateway,
  logger,
}: {
  sub: Subscription;
  gateway: PaymentGateway;
  logger: Logger;
}): Promise<Receipt> {
  logger.info(`charging ${sub.id}`);
  return gateway.charge(sub.priceCents);
}
```

## Change risk

- duplicated business rules
- boolean selectors and repeated type switches
- dead code, stale fallbacks, or speculative abstractions
- invalid states representable without validation
- partial writes without recovery or idempotency

## Verification

- missing boundary and failure scenarios
- tests coupled to implementation details
- nondeterministic tests or timing sleeps
- swallowed errors and unobservable background failures

Before reporting a smell, trace its consequence. Prefer a focused remedy over a broad rewrite.
