# Object–Relational Behavior Patterns

Use these only where object identity and transactional persistence require them.


## Unit of Work

**Problem:** write a consistent set of object changes. **Use when:** one business transaction loads and modifies multiple objects. **Tradeoff:** hidden tracking and flush order complicate failures and debugging. **Review:** identify the owner, commit boundary, ordering, rollback, and retry behavior.

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

**Problem:** prevent contradictory in-memory copies of one database row. **Use when:** object identity matters within a request or transaction. **Tradeoff:** scope leaks cause stale data and memory growth. **Review:** verify one explicit scope and eviction at its end; do not confuse identity reuse with a cross-request cache.

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

**Problem:** avoid fetching related data before it is needed. **Use when:** access is optional and a live loading context is reliable. **Tradeoff:** access can trigger hidden I/O, N+1 queries, or failures after the session closes. **Review:** inspect query counts and lifetime at every lazy access; prefer explicit eager/batched loading on known paths.

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
