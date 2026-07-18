# Objects and Responsibilities

## Behavior and data ownership

- Give each class or module one cohesive responsibility behind a small interface.
- Group code that changes for the same reason; separate work that changes independently.
- Keep behavior near the data and rules it uses, unless that would couple domain code to presentation or infrastructure.
- Keep configuration, state, and behavior with the layer that owns the decision.
- Expose behavior through narrow capabilities; do not make callers navigate collaborator internals or depend on an object graph's shape.
- Put dependencies at construction or explicit call boundaries instead of fetching hidden globals.
- Keep public APIs smaller than implementations and expose only stable concepts.
- Accept a boundary only when it protects an invariant, clarifies ownership, translates representations, adds policy, or isolates change. Delete pass-through layers otherwise.

## Valid states and encapsulation

- Make invalid states hard to construct. Validate untrusted input at creation or system boundaries, then trust the validated domain type internally.
- Hide mutable state behind operations that preserve invariants.
- Do not expose mutable internals through public fields, getters, collections, or shared references; return immutable values or defensive copies where needed.
- Treat encapsulation as control over valid operations, not as automatic getters and setters.
- Make delivery semantics explicit for event-driven state changes: define ordering, retries, duplicate handling, partial failure, and observability.
- Validate deployment configuration at startup, secure secrets, and make effective non-secret values observable.

## Cohesion and composition

- Prefer composition, delegation, interfaces, components, or mixins when inheritance would exist only for code reuse.
- Use inheritance only for a stable, genuine subtype relationship where callers can safely substitute every subtype for its base.
- Ensure base abstractions do not depend on concrete derivatives.
- Avoid vague `Manager`, `Service`, and `Utils` containers that accumulate unrelated behavior.
- Couple to stable capabilities rather than concrete organization; minimize how much each component knows about collaborators.
- Use events when multiple independent consumers need notification, but keep direct calls when their ordering and atomicity are part of the producer's operation.
- Choose callbacks, streams, queues, or event logs according to delivery and ordering requirements; decoupling does not erase operational failures.
- Use transformation pipelines when work naturally flows from input to output. Keep intermediate representations explicit and transformations composable.
- Externalize values that legitimately vary by environment or deployment; do not turn every constant into configuration or create another language to maintain.

## Objects, values, and records

- Use transparent, preferably immutable records for values and transformations that need no protected identity or mutation.
- Use objects when behavior and invariants belong with hidden state.
- Prefer immutable values when identity and mutation are unnecessary.
- Translate wire, serialization, and persistence shapes at boundaries instead of letting them silently become the domain model.
- Choose representation from the behavior and invariants it must protect, not from a blanket preference for classes or records.

## Examples

### Don't: expose invalid and mutable state

```typescript
class Wallet {
  balanceCents = 0;
  transactions: Transaction[] = [];
}

wallet.balanceCents = -500;
wallet.transactions.length = 0;
```

### Do: validate operations and hide mutation

```typescript
class Wallet {
  #balanceCents = 0;
  #transactions: Transaction[] = [];

  withdraw(amountCents: number): void {
    if (amountCents <= 0 || amountCents > this.#balanceCents) {
      throw new InsufficientFundsError(amountCents, this.#balanceCents);
    }

    this.#balanceCents -= amountCents;
    this.#transactions.push({ kind: "withdrawal", amountCents });
  }

  get transactions(): readonly Transaction[] {
    return [...this.#transactions];
  }
}
```

### Don't: inherit only to reuse an implementation

```typescript
class CsvImporter extends HttpClient {
  async import(url: string): Promise<Row[]> {
    return parseCsv(await this.get(url));
  }
}
```

### Do: compose the narrow capability

```typescript
class CsvImporter {
  constructor(private readonly fetchText: (url: string) => Promise<string>) {}

  async import(url: string): Promise<Row[]> {
    return parseCsv(await this.fetchText(url));
  }
}
```

### Don't: couple callers to an object graph

```typescript
customer.getAccount().getOrders().last().getTotals().setDiscount(amount);
```

### Do: ask the owner to perform the behavior

```typescript
customer.applyDiscountToLatestOrder(amount);
```

### Don't: let an external representation become the domain model

```typescript
type Subscription = {
  sub_id: string;
  renews_ts: string;
  is_cancelled: 0 | 1;
};
```

### Do: translate and validate at the boundary

```typescript
type Subscription = {
  id: string;
  renewsAt: Date;
  status: "active" | "cancelled";
};

function toSubscription(raw: RawSubscription): Subscription {
  const renewsAt = new Date(raw.renews_ts);
  if (Number.isNaN(renewsAt.valueOf())) throw new Error("Invalid renewal date");

  return {
    id: raw.sub_id,
    renewsAt,
    status: raw.is_cancelled === 1 ? "cancelled" : "active",
  };
}
```
