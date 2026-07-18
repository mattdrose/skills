# Metadata and Query Patterns

Use these when repeated mapping or query behavior warrants an explicit abstraction.


## Metadata Mapping

**Problem:** remove repetitive hand-written mapping code. **Use when:** many mappings follow stable, uniform rules. **Tradeoff:** generic interpretation makes control flow, errors, and performance indirect. **Review:** ensure metadata is validated, debuggable, and supports exceptions without an ad hoc language.


## Query Object

**Problem:** express reusable query criteria without embedding data-source syntax in callers. **Use when:** queries are composed dynamically over a bounded vocabulary. **Tradeoff:** a home-grown query language can be leaky, unsafe, or less capable than the store. **Review:** require parameterization and verify every operation translates correctly and efficiently.

### Don't: criteria concatenated into SQL by callers

```typescript
function findInvoices(status: string, minTotalCents?: number) {
  let sql = `SELECT * FROM invoices WHERE status = '${status}'`; // injectable, unparameterized
  if (minTotalCents !== undefined) sql += ` AND total_cents > ${minTotalCents}`;
  return db.query(sql);
}
```

### Do: a parameterized query object over a bounded vocabulary

```typescript
const query = new InvoiceQuery().where("status", "=", "overdue").where("totalCents", ">", 500_00);

const { text, values } = query.toSql(); // "... WHERE status = $1 AND total_cents > $2"
const invoices = await db.query(text, values);
```


## Repository

**Problem:** give domain code collection-like access to aggregates. **Use when:** a Domain Model benefits from a storage-independent retrieval boundary. **Tradeoff:** generic repositories hide important query semantics or merely rename a DAO. **Review:** keep domain vocabulary and aggregate boundaries visible; use Query Objects for variable criteria rather than exposing storage syntax.

### Don't: a generic repository that leaks storage syntax

```typescript
interface Repository<T> {
  query(sqlWhere: string): Promise<T[]>;
}

// schema details spread to every caller
const risky = await customerRepo.query("credit_limit_cents < balance_cents");
```

### Do: a repository speaking domain vocabulary

```typescript
interface CustomerRepository {
  overCreditLimit(): Promise<Customer[]>;
  byId(id: CustomerId): Promise<Customer | undefined>;
  save(customer: Customer): Promise<void>;
}

const risky = await customers.overCreditLimit();
```
