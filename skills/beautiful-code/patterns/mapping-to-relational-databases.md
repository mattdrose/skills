# Relational Mapping

Treat object and relational models as different representations with an explicit translation policy.

## Identity

Define how database keys map to object identity. Within one unit of work, the same row should normally resolve to the same object.

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

Choose foreign keys, association tables, or dependent mappings according to ownership and cardinality. Specify cascade behavior explicitly.

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

Use single-table, class-table, or concrete-table mapping only after weighing nullability, joins, constraints, and polymorphic queries.

## Loading

Default to explicit query shapes. Lazy loading is useful only while its session lifetime and query cost remain visible.

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

Check transaction boundaries, N+1 queries, lost updates, partial writes, duplicate in-memory identities, and schema constraints that disagree with code.
