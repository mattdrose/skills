# Organizing Data

Data design should make valid states easy to construct and invalid states difficult to represent.

## Useful transformations

- Encapsulate mutable fields and collections; do not expose writable internals.
- Replace related primitives or positional arrays with named domain values.
- Replace magic literals with named constants when the name adds domain meaning.
- Replace type codes with a constrained type; use subclasses or strategies only when behavior truly varies.
- Distinguish identity-bearing entities from immutable values. Define equality accordingly.
- Prefer one source of truth; derive secondary values unless caching is necessary and synchronized.
- Make one side authoritative in bidirectional relationships and update both sides atomically.
- Replace nullable or temporary state with explicit lifecycle states when absence has meaning.

### Don't: represent a constrained concept as a raw primitive

```typescript
interface Payment {
  amount: number; // cents? dollars? negative allowed?
  currency: string; // any string passes the type checker
}

function refund(payment: Payment, amount: number) {
  payment.amount -= amount; // nothing prevents refunding more than was paid
}
```

### Do: use a constrained domain value

```typescript
class Money {
  readonly cents: number;
  readonly currency: "USD" | "EUR";

  private constructor({ cents, currency }: { cents: number; currency: "USD" | "EUR" }) {
    this.cents = cents;
    this.currency = currency;
  }

  static of({ cents, currency }: { cents: number; currency: "USD" | "EUR" }): Money {
    if (!Number.isInteger(cents) || cents < 0) throw new Error("invalid amount");
    return new Money({ cents, currency });
  }

  subtract(other: Money): Money {
    if (other.currency !== this.currency) throw new Error("currency mismatch");
    return Money.of({
      cents: this.cents - other.cents,
      currency: this.currency,
    }); // negative result rejected
  }
}
```

### Don't: expose a writable internal collection

```typescript
class Course {
  students: Student[] = []; // any caller can mutate and bypass the capacity rule

  get roster(): Student[] {
    return this.students;
  }
}

course.roster.push(extraStudent); // enrollment limit never checked
```

### Do: encapsulate the collection behind guarded operations

```typescript
class Course {
  private students: Student[] = [];

  enroll(student: Student): void {
    if (this.students.length >= this.capacity) throw new Error("course full");
    this.students.push(student);
  }

  get roster(): readonly Student[] {
    return this.students;
  }
}
```

## Migration discipline

Introduce the new representation at one boundary, migrate callers in small groups, then remove the old representation. Validate external input before constructing domain values. Preserve serialization formats, database constraints, equality, and ordering unless changing them is explicitly in scope.

## Review checks

Ask who owns mutation, where invariants are enforced, whether aliases can bypass them, and how persistence or wire formats are affected. A wrapper that merely renames a primitive without constraining or clarifying it may add ceremony rather than design value.
