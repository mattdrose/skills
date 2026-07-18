# Data Source Patterns

Choose by domain complexity and desired separation from storage.


## Table Data Gateway

**Problem:** centralize data access for a table. **Use when:** callers can work with records and SQL-oriented operations. **Tradeoff:** table shape leaks upward and domain behavior has no natural home. **Review:** check that SQL is confined and compare Row Data Gateway when per-row behavior is clearer.

### Don't: SQL scattered through callers

```typescript
async function suspendOverdueMembers(): Promise<void> {
  const rows = await db.query(
    "SELECT id FROM members WHERE balance_due_cents > 0 AND due_date < now()",
  );
  for (const row of rows) {
    await db.query("UPDATE members SET status = 'suspended' WHERE id = $1", [row.id]);
  }
}
```

### Do: confine SQL to a Table Data Gateway

```typescript
class MemberGateway {
  findOverdue(): Promise<MemberRow[]> {
    return db.query("SELECT id FROM members WHERE balance_due_cents > 0 AND due_date < now()");
  }

  suspend(id: string): Promise<void> {
    return db.query("UPDATE members SET status = 'suspended' WHERE id = $1", [id]);
  }
}

async function suspendOverdueMembers(members: MemberGateway): Promise<void> {
  for (const row of await members.findOverdue()) {
    await members.suspend(row.id);
  }
}
```


## Row Data Gateway

**Problem:** give each loaded row a persistence interface. **Use when:** records are simple and row identity is central. **Tradeoff:** objects couple row shape to persistence and can scatter queries. **Review:** look for batched access and explicit transaction ownership; compare Table Data Gateway for set operations.


## Active Record

**Problem:** combine a domain-shaped record with its persistence. **Use when:** domain logic is modest and closely follows the schema. **Tradeoff:** complex rules and persistence independence degrade quickly. **Review:** if mapping or business behavior is becoming elaborate, prefer Data Mapper plus Domain Model; distinguish it from Row Data Gateway by its domain behavior.


## Data Mapper

**Problem:** persist objects without making them know storage. **Use when:** a rich domain model must remain persistence-independent. **Tradeoff:** identity, change tracking, and mapping add substantial machinery. **Review:** ensure storage concepts do not leak into domain objects and that the complexity is justified over Active Record.

### Don't: an Active Record accumulating complex rules

```typescript
class Invoice extends ActiveRecord {
  async finalize(): Promise<void> {
    this.totalCents = this.lines.reduce((sum, l) => sum + l.amountCents, 0);
    if ((await this.customer()).isDelinquent) this.totalCents *= 1.02; // rules now need a live DB
    await this.save();
    await (await this.customer()).recalculateCredit();
  }
}
```

### Do: a Domain Model persisted by a Data Mapper

```typescript
class Invoice {
  finalize(customer: Customer): void {
    this.total = this.lines.reduce((sum, l) => sum.add(l.amount), Money.zero("USD"));
    if (customer.isDelinquent) this.total = this.total.times(1.02);
  }
}

class InvoiceMapper {
  async save(invoice: Invoice): Promise<void> {
    await db.query("UPDATE invoices SET total_cents = $1 WHERE id = $2", [
      invoice.total.cents,
      invoice.id,
    ]);
  }
}
```
