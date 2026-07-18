# Data Access

Choose by domain complexity and the separation storage actually requires.

## Table Data Gateway

**Problem:** Centralize access to a table. **Use when:** callers naturally use records and SQL-oriented set operations. **Trade-off:** table shape leaks upward and behavior has no natural home. **Review:** confine SQL; compare Row Data Gateway when per-row behavior is clearer. Scattered SQL causes inconsistent policy and transaction handling.

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

**Problem:** Give each loaded row a persistence interface. **Use when:** records are simple and row identity is central. **Trade-off:** row shape and persistence remain coupled, and per-row operations can create scattered or N+1 queries. **Review:** inspect batching and explicit transaction ownership; prefer Table Data Gateway for set operations.

## Active Record

**Problem:** Combine a domain-shaped record with persistence. **Use when:** business behavior is modest and closely follows the schema. **Trade-off:** complex rules become database-dependent and persistence independence degrades quickly. **Review:** distinguish it from Row Data Gateway by its domain behavior; move to Domain Model plus Data Mapper when mapping or interacting rules become elaborate.

## Data Mapper

**Problem:** Persist objects without making them know storage. **Use when:** a rich Domain Model must stay persistence-independent. **Trade-off:** identity, change tracking, mapping, and orchestration add substantial machinery. **Review:** prevent storage concepts from leaking into domain objects and verify this cost is earned over Active Record. Keep transaction ownership and partial-write behavior explicit.

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
