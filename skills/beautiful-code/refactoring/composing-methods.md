# Composing Methods

Functions should present one coherent idea at a consistent level of abstraction.

## Common transformations

- **Extract function:** name a meaningful step, isolate a rule, or remove duplication.
- **Inline function:** remove indirection when the body communicates as clearly as the name.
- **Replace temporary with query:** expose how a value is derived when repeated computation is safe and clear.
- **Split variable:** use separate names when one variable represents different meanings over time.
- **Avoid parameter assignment:** create a local value so inputs retain a stable meaning.
- **Introduce function object:** use only when a genuinely complex computation needs shared intermediate state.
- **Substitute algorithm:** replace convoluted mechanics with an equivalent, clearer approach.

### Don't: mix levels of abstraction in one function

```typescript
function printOwing(invoice: Invoice): string {
  let outstanding = 0;
  for (const order of invoice.orders) {
    outstanding += order.amount; // low-level accumulation next to formatting and policy
  }
  const dueDate = new Date(invoice.issuedAt.getTime() + 30 * 24 * 60 * 60 * 1000);
  return `Customer: ${invoice.customer}\nOwed: ${outstanding}\nDue: ${dueDate.toISOString()}`;
}
```

### Do: extract named steps that state intent

```typescript
function printOwing(invoice: Invoice): string {
  return renderStatement({
    customer: invoice.customer,
    owed: outstandingAmount(invoice),
    due: dueDate(invoice),
  });
}

const outstandingAmount = (invoice: Invoice) =>
  invoice.orders.reduce((sum, order) => sum + order.amount, 0);

const dueDate = (invoice: Invoice) =>
  new Date(invoice.issuedAt.getTime() + 30 * 24 * 60 * 60 * 1000);

const renderStatement = ({ customer, owed, due }: { customer: string; owed: number; due: Date }) =>
  `Customer: ${customer}\nOwed: ${owed}\nDue: ${due.toISOString()}`;
```

### Don't: reuse one variable for different meanings

```typescript
function describeField(height: number, width: number): string {
  let temp = 2 * (height + width); // temp is the perimeter here
  const perimeterNote = `perimeter: ${temp}`;
  temp = height * width; // now temp means area
  return `${perimeterNote}, area: ${temp}`;
}
```

### Do: give each meaning its own name

```typescript
function describeField({ height, width }: { height: number; width: number }): string {
  const perimeter = 2 * (height + width);
  const area = height * width;
  return `perimeter: ${perimeter}, area: ${area}`;
}
```

## Extraction checklist

Choose a name that states intent, not mechanics. Determine inputs, outputs, mutations, errors, and timing. If extraction requires many parameters or multiple mutable outputs, first reconsider ownership or split the operation differently.

Do not extract merely to shorten a function. Tiny helpers with weak names can scatter a simple flow. Keep code inline when its context is essential and the resulting function already reads clearly.

## Review standard

After refactoring, the top-level function should describe the workflow while lower-level functions explain individual decisions. Side effects should be visible, dependencies explicit, and temporary values should have one meaning each.
