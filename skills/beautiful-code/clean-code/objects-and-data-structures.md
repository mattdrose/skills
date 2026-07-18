# Objects and Data

Choose representation according to the behavior the code must protect.

- Use value-like data structures for transparent records and transformations.
- Use objects when behavior and invariants belong with hidden state.
- Do not expose mutable internals through getters, collections, or references.
- Make invalid states hard to construct; validate at creation and trust the type internally.
- Avoid long navigation chains that make callers depend on an object graph's shape.
- Put behavior near the data and rules it uses, unless doing so would couple the domain to
  presentation or infrastructure.
- Prefer immutable values where identity and mutation are unnecessary.
- Keep serialization and persistence shapes from silently becoming the domain model.

Encapsulation is control over valid operations, not the automatic addition of getters and setters.

## Examples

### Don't: expose mutable internals and allow invalid states

```typescript
class Wallet {
  balanceCents = 0;
  transactions: Transaction[] = [];
}

wallet.balanceCents = -500; // invalid state is trivially constructible
wallet.transactions.length = 0; // history erased from outside the class
```

### Do: validate at the boundary and hide mutation behind operations

```typescript
class Wallet {
  #balanceCents = 0;
  #transactions: Transaction[] = [];

  withdraw(amountCents: number): void {
    if (amountCents <= 0 || amountCents > this.#balanceCents) {
      throw new InsufficientFundsError(amountCents, this.#balanceCents);
    }

    this.#balanceCents -= amountCents;
    this.#transactions.push({
      kind: "withdrawal",
      amountCents,
      at: new Date(),
    });
  }

  get transactions(): readonly Transaction[] {
    return this.#transactions;
  }
}
```

### Don't: let the wire format become the domain model

```typescript
// the API's raw shape flows through the whole app
type Subscription = {
  sub_id: string;
  renews_ts: string; // ISO string parsed ad hoc wherever it's needed
  is_cancelled: 0 | 1;
};

const active = subs.filter((s) => s.is_cancelled !== 1 && Date.parse(s.renews_ts) > Date.now());
```

### Do: translate into a domain type at the boundary

```typescript
type Subscription = {
  id: string;
  renewsAtMs: number;
  status: "active" | "cancelled";
};

function toSubscription(raw: RawSubscription): Subscription {
  return {
    id: raw.sub_id,
    renewsAtMs: Date.parse(raw.renews_ts),
    status: raw.is_cancelled === 1 ? "cancelled" : "active",
  };
}
```
