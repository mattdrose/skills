# Reshaping Inheritance

Inheritance is appropriate when subtypes honor the parent contract and share a stable conceptual relationship. Prefer composition when variation is independent, optional, or likely to combine.

## Transformations

- Pull shared fields or behavior upward only when they have the same meaning for every subtype.
- Push behavior downward when only specific variants support it.
- Extract a subtype when a coherent subset has distinct behavior and callers benefit from the distinction.
- Extract a superclass or interface when multiple types already share a meaningful contract.
- Collapse a hierarchy that no longer represents useful variation.
- Form a template method when an algorithm is stable but a few explicit steps vary.
- Replace inheritance with delegation when subclasses reuse implementation but cannot honor the full contract.
- Replace delegation with inheritance only when the delegate is genuinely substitutable and the complete interface should be exposed.

### Don't: inherit for reuse when the contract cannot be honored

```typescript
class CustomerList extends Array<Customer> {
  addCustomer(customer: Customer): void {
    if (this.some((c) => c.id === customer.id)) throw new Error("duplicate");
    this.push(customer);
  }
  // callers can still use push/splice and bypass the invariant
}
```

### Do: replace inheritance with delegation

```typescript
class CustomerList {
  private customers: Customer[] = [];

  add(customer: Customer): void {
    if (this.customers.some((c) => c.id === customer.id)) throw new Error("duplicate");
    this.customers.push(customer);
  }

  [Symbol.iterator]() {
    return this.customers[Symbol.iterator]();
  }
}
```

### Don't: duplicate a stable algorithm across subtypes

```typescript
class ResidentialBill {
  statement(): string {
    const base = this.units * 0.1; // same shape as CommercialBill.statement
    const taxed = base * 1.05;
    return `Residential: ${taxed.toFixed(2)}`;
  }
}

class CommercialBill {
  statement(): string {
    const base = this.units * 0.2 + 50;
    const taxed = base * 1.05;
    return `Commercial: ${taxed.toFixed(2)}`;
  }
}
```

### Do: form a template method around the varying steps

```typescript
abstract class Bill {
  statement(): string {
    const taxed = this.baseCharge() * 1.05;
    return `${this.label()}: ${taxed.toFixed(2)}`;
  }
  protected abstract baseCharge(): number;
  protected abstract label(): string;
}

class ResidentialBill extends Bill {
  protected baseCharge(): number {
    return this.units * 0.1;
  }
  protected label(): string {
    return "Residential";
  }
}
```

## Review checks

- Can every subtype be used wherever the parent is expected without surprises?
- Are shared members meaningful rather than merely structurally identical?
- Do overrides preserve invariants, errors, and side-effect expectations?
- Is subtype selection stable, or would a strategy be easier to change and combine?
- Does the hierarchy reduce duplication without coupling unrelated change axes?

Avoid speculative base classes and marker interfaces. A little duplication is cheaper than the wrong abstraction; extract shared structure after the relationship is demonstrated.
