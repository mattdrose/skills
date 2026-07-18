# Relational Structure

Make identity, relationships, value storage, and inheritance mappings explicit.

## Identity Field

**Problem:** Connect an object to its database row. **Use when:** updates or relationships require stable database identity. **Trade-off:** persistence identity leaks into the model and unsaved objects need special handling. **Review guidance:** check generation, equality, collision, and temporary-ID semantics.

## Foreign Key Mapping

**Problem:** Persist an object reference through a foreign key. **Use when:** a relationship has a natural owning side. **Trade-off:** optionality, loading, and cascading can become inconsistent. **Review guidance:** verify ownership, constraints, delete behavior, and query count; do not duplicate referenced fields where they can drift.

### Don't: copy the referenced row's fields

```typescript
interface OrderRow {
  id: string;
  customerName: string; // duplicated from customers; drifts on every rename
  customerEmail: string;
  totalCents: number;
}
```

### Do: reference through a foreign key

```typescript
interface OrderRow {
  id: string;
  customerId: string; // FK to customers.id; name and email have one home
  totalCents: number;
}

const order = await db.orders.byId(orderId);
const customer = await db.customers.byId(order.customerId);
```

## Association Table Mapping

**Problem:** Persist many-to-many relationships. **Use when:** neither side can hold one foreign key. **Trade-off:** joins and lifecycle updates cost more, and duplicate links are possible. **Review guidance:** require a uniqueness constraint and explicit add/remove ownership.

## Dependent Mapping

**Problem:** Persist a child with no independent lifecycle. **Use when:** its owner exclusively creates, addresses, and deletes it. **Trade-off:** later sharing or independent access requires redesign. **Review guidance:** confirm exclusive ownership and transactional cascades; otherwise use an identity-bearing mapping.

## Embedded Value

**Problem:** Store a small Value Object in owner columns. **Use when:** it is queried with its owner and has no identity. **Trade-off:** flattening and repeated columns complicate evolution. **Review guidance:** preserve value equality and atomic replacement; use Serialized LOB only when fields need not be queried. Do not bury queryable fields in a blob.

### Don't: serialize queryable fields into a blob

```typescript
interface CustomerRow {
  id: string;
  address_json: string; // "customers in Ohio" now requires scanning JSON
}

await db.customers.insert({ id, address_json: JSON.stringify(address) });
```

### Do: embed the value in the owner's columns

```typescript
interface CustomerRow {
  id: string;
  address_street: string;
  address_city: string;
  address_state: string; // indexable: WHERE address_state = 'OH'
}

await db.customers.insert({ id, ...toAddressColumns(address) });
```

## Serialized LOB

**Problem:** Store an object graph as one opaque field. **Use when:** internals need neither querying nor independent updates. **Trade-off:** partial updates, indexing, compatibility, and migrations are difficult. **Review guidance:** require format versioning, migration policy, and size limits; choose Embedded Value for queryable fields.

## Single Table Inheritance

**Problem:** Map a hierarchy without joins. **Use when:** subtype count and fields are modest and polymorphic reads are common. **Trade-off:** null-heavy rows and weak subtype constraints accumulate. **Review guidance:** validate discriminators and subtype-required fields; reject unknown types rather than silently constructing the wrong subtype.

### Don't: an unchecked discriminator with implicit nulls

```typescript
function loadAccount(row: AccountRow): Account {
  if (row.type === "savings") return new SavingsAccount(row.id, row.interest_rate!);
  // unknown types silently become checking accounts with a null overdraft
  return new CheckingAccount(row.id, row.overdraft_cents!);
}
```

### Do: validate the discriminator and subtype-required fields

```typescript
function loadAccount(row: AccountRow): Account {
  switch (row.type) {
    case "savings":
      return new SavingsAccount(row.id, required(row.interest_rate, "interest_rate"));
    case "checking":
      return new CheckingAccount(row.id, required(row.overdraft_cents, "overdraft_cents"));
    default:
      throw new Error(`unknown account type: ${row.type}`);
  }
}
```

## Class Table Inheritance

**Problem:** Map shared and subtype fields in normalized tables. **Use when:** subtype integrity and shared identity matter. **Trade-off:** reads and writes require joins across the hierarchy. **Review guidance:** inspect polymorphic query cost and transactional multi-table inserts; compare Single Table for read simplicity.

## Concrete Table Inheritance

**Problem:** Persist each concrete subtype independently. **Use when:** subtype-specific access dominates and base instances do not exist. **Trade-off:** shared columns and constraints drift, while polymorphic queries require UNIONs. **Review guidance:** avoid it when cross-subtype identity or queries are frequent.

## Inheritance Mappers

**Problem:** Share Data Mapper behavior across a hierarchy. **Use when:** related mappers genuinely repeat identity or field logic. **Trade-off:** mapper inheritance can amplify a fragile domain hierarchy. **Review guidance:** verify shared behavior and complete subtype dispatch; prefer composition for incidental reuse.
