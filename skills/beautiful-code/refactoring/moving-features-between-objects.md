# Moving Responsibilities

Place behavior with the data, invariants, or lifecycle it primarily serves. Good ownership reduces navigation, leakage, and coordinated edits.

## Transformations

- **Move function or field:** relocate behavior or state to its natural owner; keep a temporary delegate if callers need gradual migration.
- **Extract component:** split a unit with distinct responsibilities or independent reasons to change.
- **Inline component:** merge a type whose boundary adds no useful meaning or isolation.
- **Hide delegate:** expose an intention-level operation instead of making callers traverse internals.
- **Remove middleman:** let callers use the real collaborator when delegation is the layer’s only purpose.
- **Add a local adapter:** wrap an external type when required behavior cannot be added safely upstream.

### Don't: make callers traverse internals

```typescript
class Customer {
  constructor(public department: Department) {}
}

// caller must know Customer -> Department -> Employee navigation
const approver = customer.department.manager;
if (customer.department.plan.tier === "enterprise") notify(approver);
```

### Do: hide the delegate behind an intention-level operation

```typescript
class Customer {
  constructor(private department: Department) {}

  get approver(): Employee {
    return this.department.manager;
  }

  isEnterprise(): boolean {
    return this.department.plan.tier === "enterprise";
  }
}

if (customer.isEnterprise()) notify(customer.approver);
```

### Don't: let one class carry two independent responsibilities

```typescript
class Person {
  name = "";
  officeAreaCode = ""; // telephone details change for different reasons than identity
  officeNumber = "";

  get telephoneNumber(): string {
    return `(${this.officeAreaCode}) ${this.officeNumber}`;
  }
}
```

### Do: extract a component with its own reason to change

```typescript
class TelephoneNumber {
  readonly areaCode: string;
  readonly number: string;

  constructor({ areaCode, number }: { areaCode: string; number: string }) {
    this.areaCode = areaCode;
    this.number = number;
  }

  toString(): string {
    return `(${this.areaCode}) ${this.number}`;
  }
}

class Person {
  readonly name: string;
  readonly officePhone: TelephoneNumber;

  constructor({ name, officePhone }: { name: string; officePhone: TelephoneNumber }) {
    this.name = name;
    this.officePhone = officePhone;
  }
}
```

## Choosing an owner

Consider which unit owns the invariant, contains most required data, changes for the same reasons, and can offer the narrowest stable contract. Avoid moving behavior merely to eliminate imports; coupling can be hidden rather than reduced.

## Review checks

- Does the move reduce knowledge between modules?
- Are mutation and transaction boundaries still correct?
- Did visibility become broader only to enable the move?
- Are cycles, duplicate sources of truth, or pass-through APIs introduced?
- Is compatibility delegation temporary and clearly bounded?
