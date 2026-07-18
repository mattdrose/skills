# Object–Relational Structure Patterns

Use these to make identity, relationships, value storage, and inheritance mappings explicit.


## Identity Field

**Problem:** connect an object to its database row. **Use when:** updates or relationships require stable database identity. **Tradeoff:** persistence identity leaks into the object model and unsaved objects need special handling. **Review:** check generation, equality, and temporary-ID semantics.


## Foreign Key Mapping

**Problem:** persist an object reference through a foreign key. **Use when:** a relationship has a natural owning side. **Tradeoff:** optionality, loading, and cascading can become inconsistent. **Review:** verify ownership, constraints, delete behavior, and query count.

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

**Problem:** persist many-to-many relationships. **Use when:** neither side can hold a single foreign key. **Tradeoff:** joins and lifecycle updates cost more, and duplicate links are possible. **Review:** require a uniqueness constraint and explicit add/remove ownership.


## Dependent Mapping

**Problem:** persist a child with no independent lifecycle. **Use when:** the owner exclusively creates, addresses, and deletes it. **Tradeoff:** later sharing or independent access requires redesign. **Review:** confirm exclusive ownership and transactional cascade behavior; otherwise use an identity-bearing relationship mapping.


## Embedded Value

**Problem:** store a small value object in its owner's columns. **Use when:** the value is queried with its owner and has no identity. **Tradeoff:** column duplication and flattening complicate evolution. **Review:** preserve value equality and atomic replacement; compare Serialized LOB when fields need not be queried.

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

**Problem:** store an object graph as one opaque field. **Use when:** its internals need neither querying nor independent updates. **Tradeoff:** partial updates, indexing, compatibility, and migrations are difficult. **Review:** demand format versioning and size limits; choose Embedded Value for queryable fields.


## Single Table Inheritance

**Problem:** map a class hierarchy without joins. **Use when:** subtype count and fields are modest and polymorphic reads are common. **Tradeoff:** null-heavy rows and weak subtype constraints accumulate. **Review:** validate discriminators and subtype-required fields; compare Class Table Inheritance when integrity matters more than read simplicity.

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

**Problem:** map shared and subtype fields in normalized tables. **Use when:** subtype integrity and shared identity matter. **Tradeoff:** reads and writes require joins across the hierarchy. **Review:** inspect polymorphic query cost and transactional inserts; compare Single Table for read simplicity.


## Concrete Table Inheritance

**Problem:** persist each concrete subtype independently. **Use when:** subtype-specific access dominates and base instances do not exist. **Tradeoff:** shared columns, constraints, and polymorphic queries are duplicated. **Review:** check schema drift and UNION cost; avoid when cross-subtype identity or queries are frequent.


## Inheritance Mappers

**Problem:** share mapper behavior across an inheritance hierarchy. **Use when:** Data Mappers repeat identity or field logic for related types. **Tradeoff:** mapper inheritance can mirror and amplify a fragile domain hierarchy. **Review:** verify shared behavior is real and subtype dispatch is complete; composition may be clearer for incidental reuse.
