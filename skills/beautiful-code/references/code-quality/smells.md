# Code Smells

Treat a smell as a prompt to investigate, not proof that code is wrong. Trace a concrete consequence, identify the change pressure, and judge the code in context.

## Intent

- Replace misleading, inconsistent, vague, or unexplained names with domain language.
- Name magic values, units, and business rules.
- Simplify dense expressions and keep each section at one level of abstraction.
- Improve code instead of adding comments that compensate for unclear intent.
- Delete dead code, stale fallbacks, and abstractions with no current use.
- Remove middlemen that delegate without adding boundary value.

**Don't:**

```typescript
if (user.status === 3 && Date.now() - user.createdAtMs > 2592000000) {
  applyDiscount(user, 0.15);
}
```

**Do:**

```typescript
const LOYALTY_DISCOUNT_RATE = 0.15;
const THIRTY_DAYS_MS = 30 * 24 * 60 * 60 * 1000;
const isEstablishedMember =
  user.status === UserStatus.Active && Date.now() - user.createdAtMs > THIRTY_DAYS_MS;

if (isEstablishedMember) applyDiscount(user, LOYALTY_DISCOUNT_RATE);
```

## Responsibility

- Split functions that mix unrelated phases or abstraction levels; first consider simplifying the algorithm.
- Split modules and classes that change for unrelated reasons.
- Move behavior toward the data and rules it governs when another unit knows their internals better than their owner.
- Keep high-level policy out of low-level infrastructure.
- Gather values that repeatedly travel together into a meaningful domain value.
- Replace long parameter lists when they reveal a missing concept or excessive coupling; do not hide unrelated inputs in a bag.
- Model temporary fields or unclear lifecycle states as explicit phases or separate objects.
- Consolidate repeated branching by kind when adding a kind requires finding every switch. Use a strategy, lookup, or polymorphism only when the variation is stable enough to justify it.
- Replace inheritance when a subtype cannot honor the parent's expectations; prefer composition.

**Don't:**

```typescript
function shipOrder(order: Order, street: string, city: string, postal: string, country: string) {}
```

**Do:**

```typescript
interface Address {
  street: string;
  city: string;
  postal: string;
  country: string;
}

function shipOrder({ order, destination }: { order: Order; destination: Address }) {}
```

**Don't:**

```typescript
class OrderReport {
  format(order: Order): string {
    const subtotalCents = order.lines.reduce(
      (sum, line) => sum + line.priceCents * line.quantity,
      0,
    );
    const taxCents = Math.round((subtotalCents * order.customer.region.taxBasisPoints) / 10_000);
    return `${order.id}: ${formatCents(subtotalCents + taxCents)}`;
  }
}
```

**Do:**

```typescript
class Order {
  totalWithTaxCents(): number {
    const subtotalCents = this.lines.reduce(
      (sum, line) => sum + line.priceCents * line.quantity,
      0,
    );
    const taxCents = Math.round((subtotalCents * this.customer.region.taxBasisPoints) / 10_000);
    return subtotalCents + taxCents;
  }
}

class OrderReport {
  format(order: Order): string {
    return `${order.id}: ${formatCents(order.totalWithTaxCents())}`;
  }
}
```

**Don't:**

```typescript
function deliveryDays(method: ShippingMethod): number {
  switch (method.kind) {
    case "standard":
      return 5;
    case "express":
      return 2;
    case "overnight":
      return 1;
  }
}
```

**Do:**

```typescript
const deliveryDays: Record<ShippingMethod["kind"], number> = {
  standard: 5,
  express: 2,
  overnight: 1,
};
```

## Coupling

- Replace long object-navigation chains with intention-level operations. Do not make callers depend on an internal graph's shape.
- Pass dependencies explicitly instead of retrieving hidden globals or services.
- Break dependency cycles and keep base modules unaware of concrete implementations.
- Narrow broad interfaces to what consumers need.
- Prevent vendor types and infrastructure details from leaking into core policy.
- Reduce inappropriate intimacy: stop modules from reaching into each other's internals; narrow the interface or move ownership.
- Keep useful boundaries. Remove indirection only when it contributes no policy, isolation, translation, or stability.

**Don't:**

```typescript
function chargeSubscription(subscription: Subscription): Promise<Receipt> {
  const gateway = ServiceLocator.get("payments");
  globalThis.appLogger.info(`charging ${subscription.id}`);
  return gateway.charge(subscription.priceCents);
}
```

**Do:**

```typescript
function chargeSubscription({
  subscription,
  gateway,
  logger,
}: {
  subscription: Subscription;
  gateway: PaymentGateway;
  logger: Logger;
}): Promise<Receipt> {
  logger.info(`charging ${subscription.id}`);
  return gateway.charge(subscription.priceCents);
}
```

## Change risk

- Centralize duplicated knowledge so fixes cannot diverge; do not abstract merely similar syntax that may evolve independently.
- Move a scattered rule to one owning boundary when a single change requires edits across many units.
- Separate responsibilities when one unit changes for unrelated concerns.
- Replace boolean selectors with intention-revealing operations or distinct behavior.
- Constrain primitive values when they permit invalid states; validate at trust boundaries.
- Protect multi-step writes with transactions, recovery, or idempotency. Partial writes must not silently leave inconsistent state.
- Remove speculative flexibility until a real use requires it.
- Preserve behavior while restructuring; broad rewrites amplify risk and obscure the cause of failures.

**Don't:**

```typescript
function invoiceTotalCents(lines: Line[]): number {
  const DISCOUNT_THRESHOLD_CENTS = 100_000;
  const DISCOUNT_BASIS_POINTS = 500;
  const subtotalCents = lines.reduce((sum, line) => sum + line.priceCents * line.qty, 0);
  return subtotalCents > DISCOUNT_THRESHOLD_CENTS
    ? Math.round((subtotalCents * (10_000 - DISCOUNT_BASIS_POINTS)) / 10_000)
    : subtotalCents;
}

function quoteTotalCents(lines: Line[]): number {
  const DISCOUNT_THRESHOLD_CENTS = 100_000;
  const DISCOUNT_BASIS_POINTS = 500;
  const subtotalCents = lines.reduce((sum, line) => sum + line.priceCents * line.qty, 0);
  return subtotalCents > DISCOUNT_THRESHOLD_CENTS
    ? Math.round((subtotalCents * (10_000 - DISCOUNT_BASIS_POINTS)) / 10_000)
    : subtotalCents;
}
```

**Do:**

```typescript
const DISCOUNT_THRESHOLD_CENTS = 100_000;
const DISCOUNT_BASIS_POINTS = 500;

function totalCents(lines: Line[]): number {
  const subtotalCents = lines.reduce((sum, line) => sum + line.priceCents * line.qty, 0);
  return subtotalCents > DISCOUNT_THRESHOLD_CENTS
    ? Math.round((subtotalCents * (10_000 - DISCOUNT_BASIS_POINTS)) / 10_000)
    : subtotalCents;
}

const invoiceTotalCents = totalCents;
const quoteTotalCents = totalCents;
```

## Verification

- Cover boundaries, failures, state transitions, side effects, and important regressions.
- Test observable behavior through stable boundaries, not private structure or call sequences.
- Control time, randomness, concurrency, and external state; never synchronize with arbitrary sleeps.
- Surface swallowed errors and background failures with actionable context, without exposing secrets or sensitive data.
- Treat invalid-state construction, partial persistence, and unobserved failure as correctness and security risks, not merely style concerns.
- Demand evidence that a proposed refactor preserves behavior. A passing narrow test does not establish system-wide safety.

## Choosing a response

1. State the concrete maintenance, correctness, security, or failure-recovery problem.
2. Decide whether the signal is local or evidence of misplaced ownership.
3. Identify the next likely change and the pressure it places on the design.
4. Choose the smallest focused transformation that removes that pressure.
5. Verify behavior before and after the change.
6. Leave the code alone when the response would add more complexity than the smell costs.

Prefer deletion, renaming, moving responsibility, narrowing an interface, or consolidating one rule over a broad rewrite. Introduce a domain type, strategy, or new boundary only when it makes invalid states harder, ownership clearer, or the next likely change easier.
