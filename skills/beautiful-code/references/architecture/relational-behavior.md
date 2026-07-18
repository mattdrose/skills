# Relational Behavior

Use these patterns only where object identity and transactional persistence require them.

## Unit of Work

**Problem:** Write a consistent set of object changes. **Use when:** one business transaction loads and modifies multiple objects. **Trade-off:** hidden tracking and flush order complicate debugging, retries, and failures. **Review guidance:** identify the owner, commit boundary, write ordering, rollback, and retry behavior. Scattered saves can leave a transfer or other invariant half-applied; register changes and commit them in one database transaction.

### Don't: scattered saves with no commit boundary

```typescript
async function transferStock(from: Warehouse, to: Warehouse, sku: string, qty: number) {
  from.remove(sku, qty);
  await warehouseMapper.save(from);
  to.add(sku, qty);
  await warehouseMapper.save(to); // a failure here leaves stock half-moved
}
```

### Do: register changes, commit once

```typescript
async function transferStock({
  uow,
  from,
  to,
  sku,
  qty,
}: {
  uow: UnitOfWork;
  from: Warehouse;
  to: Warehouse;
  sku: string;
  qty: number;
}) {
  from.remove({ sku, qty });
  to.add({ sku, qty });
  uow.registerDirty(from);
  uow.registerDirty(to);
  await uow.commit(); // both writes in one transaction, or neither
}
```

## Identity Map

**Problem:** Prevent contradictory in-memory copies of one row. **Use when:** object identity matters within a request or transaction. **Trade-off:** scopes that leak cause stale data and unbounded memory growth. **Review guidance:** give the map one explicit Unit of Work scope and discard it at the end; do not confuse identity reuse with a cross-request cache.

### Don't: an identity map that outlives its request

```typescript
const identityMap = new Map<string, Customer>(); // module-level: stale entries, unbounded growth

async function loadCustomer(id: string): Promise<Customer> {
  const cached = identityMap.get(id);
  if (cached) return cached;
  const customer = hydrate(await db.customers.byId(id));
  identityMap.set(id, customer);
  return customer;
}
```

### Do: scope the map to one unit of work

```typescript
class CustomerMapper {
  constructor(private readonly uow: UnitOfWork) {}

  async findById(id: string): Promise<Customer> {
    const cached = this.uow.identityMap.get("customers", id);
    if (cached) return cached;

    const customer = hydrate(await db.customers.byId(id));
    this.uow.identityMap.put("customers", id, customer);
    return customer; // map is discarded when the unit of work ends
  }
}
```

## Lazy Load

**Problem:** Avoid fetching related data until needed. **Use when:** access is optional and a live loading context is reliable. **Trade-off:** ordinary access can trigger hidden I/O, N+1 queries, or failure after the session closes. **Review guidance:** inspect query counts and session lifetime at every lazy access; use explicit eager or batched loading on known paths.

### Don't: lazy access after the session closes

```typescript
async function getInvoicePage(id: string): Promise<InvoiceView> {
  const invoice = await withSession((s) => s.invoices.findById(id));
  // session is gone; this throws or silently reopens a connection
  return { number: invoice.number, lines: await invoice.lines };
}
```

### Do: eager-load what the path needs

```typescript
async function getInvoicePage(id: string): Promise<InvoiceView> {
  const invoice = await withSession((s) => s.invoices.findById(id, { include: ["lines"] }));
  return { number: invoice.number, lines: invoice.lines };
}
```
