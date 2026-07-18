# Coding Practice

## Developer intuition

**Review question:** Is persistent discomfort pointing to an unresolved contradiction or risk?

Pause when work feels unexpectedly difficult. Write down what is known, reduce distractions, and inspect the assumptions. Intuition is a prompt to gather evidence, not evidence by itself.

## Intentional programming

**Review question:** Can the implementation be explained as correct and the design choice be justified?

Avoid coding by habit, cargo cult, or accidental success. Work from explicit intent, verify assumptions, and understand dependencies. If a technique cannot be explained, investigate before relying on it.

## Algorithmic cost

**Review question:** How does resource use grow with realistic input and workload?

Estimate time and space complexity before optimizing. Then measure representative data, including worst credible cases. Prefer the simplest adequate algorithm and optimize only the bottleneck shown by evidence.

### Don't: hide quadratic growth in a convenient lookup

```typescript
function ordersWithKnownCustomers(orders: Order[], customers: Customer[]): Order[] {
  // every order rescans the whole customer list: fine at 100 rows, minutes at a million
  return orders.filter((order) => customers.some((c) => c.id === order.customerId));
}
```

### Do: pick a structure that matches the workload

```typescript
function ordersWithKnownCustomers({
  orders,
  customers,
}: {
  orders: Order[];
  customers: Customer[];
}): Order[] {
  const knownIds = new Set(customers.map((c) => c.id));
  return orders.filter((order) => knownIds.has(order.customerId));
}
```

## Refactoring

**Review question:** Is the design being improved continuously while behavior remains protected?

Refactor in small, reversible steps with feedback from tests and static checks. Separate structural cleanup from behavior changes when possible. Do not let urgency turn ordinary design maintenance into a risky future project.

## Tests as feedback

**Review question:** Do tests demonstrate important behavior through stable, public boundaries?

Design for testability and use tests to expose coupling, ambiguous interfaces, and edge cases. Favor high-value integration coverage over assertions tied to implementation details. A passing suite supports judgment; it does not replace it.

### Don't: assert behavior through implementation details

```typescript
it("applies the loyalty discount", () => {
  const cart = new Cart(items);
  cart.applyLoyaltyDiscount(member);
  // breaks on any internal refactor even when pricing is still correct
  expect((cart as any)._discountLines[0].strategyName).toBe("LoyaltyV2");
});
```

### Do: test the observable outcome at the public boundary

```typescript
it("applies the loyalty discount", () => {
  const cart = new Cart(items);
  cart.applyLoyaltyDiscount(member);
  expect(cart.total()).toBe(subtotal(items) * 0.9);
});
```

## Property-based testing

**Review question:** Can broad invariants be tested across generated inputs rather than a few examples?

State properties such as round-trip equivalence, ordering, conservation, or idempotency. Generate varied inputs and retain minimal failing cases as regressions. Review failures carefully: they often reveal an incomplete property or an unstated requirement.

## Security

**Review question:** Does the system minimize authority, exposure, and untrusted assumptions?

Validate at trust boundaries, authenticate identity, authorize each action, encrypt sensitive data, and patch dependencies. Keep secrets out of source and logs. Default to denial, reduce attack surface, and make security failures observable without leaking sensitive details.

### Don't: interpolate untrusted input into a query

```typescript
async function findUser(email: string) {
  // any user-supplied string becomes part of the SQL itself
  return db.query(`SELECT * FROM users WHERE email = '${email}'`);
}
```

### Do: parameterize at the trust boundary

```typescript
async function findUser(email: string) {
  return db.query("SELECT * FROM users WHERE email = $1", [email]);
}
```

## Naming

**Review question:** Do names express domain meaning, role, and scope consistently?

Use the language shared by users and maintainers. Rename misleading concepts as understanding improves. Avoid unexplained abbreviations, vague containers, type repetition, and clever wordplay; a name should reduce the need for comments.

### Don't: hide meaning behind vague names

```typescript
function proc(data: any[]): any[] {
  const tmp = data.filter((d) => d.flag2 && d.amt > 0);
  return tmp.map((d) => ({ ...d, val: d.amt * d.rt }));
}
```

### Do: name the domain concepts

```typescript
function accruedInterestForActiveLoans(loans: Loan[]): LoanWithInterest[] {
  return loans
    .filter((loan) => loan.isActive && loan.principal > 0)
    .map((loan) => ({
      ...loan,
      accruedInterest: loan.principal * loan.interestRate,
    }));
}
```
