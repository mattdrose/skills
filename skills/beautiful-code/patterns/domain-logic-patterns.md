# Domain Logic Patterns

Choose the simplest pattern that fits the rules; do not treat this catalog as a required architecture.


## Transaction Script

**Problem:** organize one business operation. **Use when:** workflows and rules are simple. **Tradeoff:** scripts duplicate rules and grow conditional tangles as complexity rises. **Review:** look for repeated rules or cross-script invariants; those suggest a Domain Model.


## Domain Model

**Problem:** represent interacting business rules and invariants. **Use when:** behavior is complex enough to justify behavior-rich domain objects. **Tradeoff:** mapping and orchestration cost more than with Transaction Script or Table Module. **Review:** verify rules live in the model while persistence, transport, and transaction control stay outside.

### Don't: rules duplicated across transaction scripts

```typescript
async function placeOrder(order: OrderRow): Promise<void> {
  if (order.totalCents > 500_00 && order.customerTier === "gold") order.totalCents *= 0.95;
  await db.orders.insert(order);
}

async function quoteOrder(order: OrderRow): Promise<number> {
  if (order.totalCents > 500_00 && order.customerTier === "gold") order.totalCents *= 0.95; // copy drifts
  return order.totalCents;
}
```

### Do: the rule lives once, in the domain model

```typescript
class Order {
  private lines: OrderLine[];
  private customer: Customer;

  constructor({ lines, customer }: { lines: OrderLine[]; customer: Customer }) {
    this.lines = lines;
    this.customer = customer;
  }

  total(): Money {
    const gross = this.lines.reduce((sum, l) => sum.add(l.amount), Money.zero("USD"));
    return this.customer.qualifiesForVolumeDiscount(gross) ? gross.times(0.95) : gross;
  }
}

const placed = await orders.save(order); // both paths call order.total()
const quoted = order.total();
```


## Table Module

**Problem:** organize logic around tabular data. **Use when:** record-set-oriented tools and a table-shaped domain dominate. **Tradeoff:** row-spanning logic is convenient, but rich object behavior and cross-table rules become awkward. **Review:** check that table boundaries match the rules; otherwise compare Transaction Script or Domain Model.


## Service Layer

**Problem:** expose a stable application-operation boundary. **Use when:** multiple clients need coordinated transactions, authorization, and domain calls. **Tradeoff:** it can become an anemic-domain dumping ground or needless pass-through layer. **Review:** expect orchestration at this boundary and core rules in the domain; remove services that only forward calls.

### Don't: a service layer that only forwards calls

```typescript
class OrderService {
  getOrder(id: string) {
    return this.repo.findById(id);
  }
  saveOrder(order: Order) {
    return this.repo.save(order);
  }
  cancelOrder(id: string) {
    return this.repo.cancel(id);
  } // pure indirection, no coordination
}
```

### Do: a service owning transaction and authorization, rules in the domain

```typescript
class OrderService {
  async cancelOrder({ user, id }: { user: User; id: string }): Promise<void> {
    await this.tx.run(async () => {
      const order = await this.repo.findById(id);
      this.authz.require(user, "order:cancel", order);
      order.cancel(); // invariant checks live on Order

      await this.repo.save(order);
    });
  }
}
```
