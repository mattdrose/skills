# Metadata and Queries

Add these abstractions only when repeated mapping or query behavior earns them.

## Metadata Mapping

**Problem:** Remove repetitive hand-written mapping. **Use when:** many mappings follow stable, uniform rules. **Trade-off:** generic interpretation makes control flow, errors, and performance indirect. **Review:** validate metadata, make execution debuggable, and support genuine exceptions without growing an ad hoc language.

## Query Object

**Problem:** Express reusable criteria without data-source syntax in callers. **Use when:** queries compose dynamically over a bounded vocabulary. **Trade-off:** a home-grown query language may be leaky, unsafe, or weaker than the store. **Review:** parameterize every value; validate fields and operators; verify each operation translates correctly and efficiently. Never concatenate untrusted criteria into SQL.

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
type InvoiceCriteria = {
  status: string;
  totalCents: number;
};
type QueryOperator = "=" | ">";

const invoiceColumns: Record<keyof InvoiceCriteria, string> = {
  status: "status",
  totalCents: "total_cents",
};

class InvoiceQuery {
  private readonly clauses: string[] = [];
  private readonly values: Array<string | number> = [];

  where<Field extends keyof InvoiceCriteria>(
    field: Field,
    operator: QueryOperator,
    value: InvoiceCriteria[Field],
  ): this {
    this.values.push(value);
    this.clauses.push(`${invoiceColumns[field]} ${operator} $${this.values.length}`);
    return this;
  }

  toSql(): { text: string; values: Array<string | number> } {
    const where = this.clauses.length ? ` WHERE ${this.clauses.join(" AND ")}` : "";
    return {
      text: `SELECT * FROM invoices${where}`,
      values: [...this.values],
    };
  }
}

const query = new InvoiceQuery().where("status", "=", "overdue").where("totalCents", ">", 50_000);

const { text, values } = query.toSql();
// text: "SELECT * FROM invoices WHERE status = $1 AND total_cents > $2"
// values: ["overdue", 50000]
const invoices = await db.query(text, values);
```

The typed API bounds fields, operators, and value types. Criteria from HTTP, JSON, or other external sources still require runtime validation before calling it.

## Repository

**Problem:** Give domain code collection-like access to aggregates. **Use when:** a Domain Model benefits from a storage-independent retrieval boundary. **Trade-off:** generic repositories hide query semantics or merely rename a DAO. **Review:** expose domain vocabulary and aggregate boundaries; use Query Objects for variable criteria rather than leaking SQL or schema details.

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
