# Relational Mapping

Treat object and relational models as different representations with explicit translation policy.

## Identity

**Problem:** Connect database keys to object identity without allowing duplicate in-memory representations to diverge. **Use when:** rows are loaded, updated, or referenced as objects. **Trade-off:** persistence identity can leak into the model, while temporary keys and stale instances complicate equality. **Review:** within one Unit of Work, one row should normally resolve to one object; review key generation, temporary IDs, equality, Identity Map scope, and last-write-wins loss.

### Don't: one row, two divergent objects

```typescript
const order = await orderMapper.findById("o-42");
const sameOrder = await orderMapper.findById("o-42"); // fresh object each call
order.markShipped();
sameOrder.applyDiscount(10);
await orderMapper.save(order);
await orderMapper.save(sameOrder); // last write silently wins
```

### Do: resolve a row to one object per unit of work

```typescript
class OrderMapper {
  async findById(id: string): Promise<Order> {
    const cached = this.unitOfWork.identityMap.get("orders", id);
    if (cached) return cached;

    const order = this.hydrate(await db.orders.byId(id));
    this.unitOfWork.identityMap.put("orders", id, order);
    return order;
  }
}
```

## Relationships

**Problem:** Preserve object relationships in foreign-key and table structures. **Use when:** objects reference, own, or collect other persisted objects. **Trade-off:** ownership, optionality, loading, and cascade semantics can disagree between code and schema. **Review:** choose Foreign Key Mapping, Association Table Mapping, or Dependent Mapping according to ownership and cardinality; specify constraints and add/remove ownership. An implicit cascade can leave orphans or delete shared data, so perform multi-write changes in one transaction.

### Don't: delete an owner with cascade behavior left implicit

```typescript
async function deleteOrder(id: string): Promise<void> {
  await db.orders.delete(id); // order_lines rows remain, pointing at a missing order
}
```

### Do: explicit cascade inside one transaction

```typescript
async function deleteOrder(id: string): Promise<void> {
  await db.tx(async (t) => {
    await t.orderLines.deleteByOrder(id);
    await t.orders.delete(id);
  });
}
```

## Inheritance

**Problem:** Represent an object inheritance hierarchy in relational tables. **Use when:** persisted polymorphism or subtype-specific fields are required. **Trade-off:** every strategy exchanges nullability and weak subtype constraints against joins, duplicated schema, identity complexity, or polymorphic query cost. **Review:** choose Single Table Inheritance, Class Table Inheritance, or Concrete Table Inheritance only after comparing those costs; validate discriminators and complete subtype dispatch.

## Loading

**Problem:** Load enough of an object graph without unnecessary work or hidden database access. **Use when:** related data has different access frequency or cost. **Trade-off:** eager loading over-fetches, while Lazy Load hides I/O, causes N+1 queries, and can fail after its session closes. **Review:** default to explicit query shapes; use Lazy Load only when access is optional, query cost is observable, and the loading session remains alive. Eager-load or batch known paths.

### Don't: hidden lazy loads on a known path

```typescript
const orders = await orderRepo.findByCustomer(customerId);
let totalCents = 0;
for (const order of orders) {
  // each access silently issues another query
  totalCents += (await order.lines).reduce((sum, l) => sum + l.amountCents, 0);
}
```

### Do: an explicit query shape for the path

```typescript
// one join, stated up front
const orders = await orderRepo.findByCustomerWithLines(customerId);
const totalCents = orders.flatMap((o) => o.lines).reduce((sum, l) => sum + l.amountCents, 0);
```

## Review

Trace reads and writes through transaction boundaries. Check N+1 queries, lost updates, partial writes, duplicate in-memory identities, explicit ownership/cascades, loading-context lifetime, and schema constraints that disagree with code.
