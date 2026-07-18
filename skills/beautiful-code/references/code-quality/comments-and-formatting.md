# Comments and Formatting

## Comments

- Write comments only for information code cannot express clearly: intent, non-obvious constraints, tradeoffs, protocol or compatibility requirements, security or performance requirements, and public API behavior callers must know.
- Explain why an apparently simpler implementation is wrong when removing the constraint would reintroduce a failure.
- Keep every comment accurate as behavior changes; verify nearby comments whenever editing code.
- Delete comments that paraphrase code, narrate mechanics, excuse unclear code, or preserve history better kept in version control or an issue tracker.
- Delete commented-out code rather than maintaining a second, untested implementation.
- Make TODOs identify a specific remaining problem and link to tracked work when appropriate; remove vague or obsolete TODOs.

## Locality and vertical structure

- Let the project formatter decide mechanical style; do not spend review effort on preferences it can enforce.
- Put important exports and orchestration before supporting details when language conventions permit.
- Keep related declarations together, separate unrelated ideas with whitespace, and use consistent ordering across similar files.
- Declare values near first use and keep scopes narrow.
- Keep each file cohesive. Split it only when distinct responsibilities can be named and changed independently.
- Avoid manual alignment and formatting that produce noisy diffs.

## Expression clarity

- Break dense expressions when intermediate names expose intent or important conditions.
- Prefer a direct, slightly longer statement when compression, nesting, or advanced language features increase total mental work.
- Judge clarity for a capable newcomer, not only for authors familiar with the idiom.
- Do not confuse fewer lines or polished formatting with simplicity. Readable code can still hide too many states, responsibilities, or interactions.
- Do not use formatting debates as substitutes for reviewing correctness, control flow, or coupling.

## Examples

### Don't: narrate mechanics while hiding the reason

```typescript
// Loop over items and add price times quantity.
let total = 0;
for (const item of order.lineItems) {
  total += item.priceCents * item.quantity;
}
```

### Do: preserve the constraint the code cannot explain

```typescript
// Keep totals in integer cents; floating-point dollars caused rounding bugs (#482).
let totalCents = 0;
for (const item of order.lineItems) {
  totalCents += item.priceCents * item.quantity;
}
```

### Don't: leave a stale comment contradicting behavior

```typescript
// Retry twice before failing.
for (let attempt = 0; attempt < 5; attempt++) {
  await send(request);
}
```

### Do: express the policy once

```typescript
const maxAttempts = 5;
for (let attempt = 0; attempt < maxAttempts; attempt++) {
  await send(request);
}
```

### Don't: hide display rules in a nested conditional

```typescript
const statusLabel = account.isClosed
  ? "Closed"
  : account.paymentDueCents > 0
    ? account.isOverdue
      ? "Payment overdue"
      : "Payment due"
    : "Current";
```

### Do: name the conditions that determine the display

```typescript
const hasPaymentDue = account.paymentDueCents > 0;
const needsUrgentPayment = hasPaymentDue && account.isOverdue;

let statusLabel = "Current";
if (account.isClosed) statusLabel = "Closed";
else if (needsUrgentPayment) statusLabel = "Payment overdue";
else if (hasPaymentDue) statusLabel = "Payment due";
```

### Don't: declare values far from their meaning

```typescript
function buildInvoice(order: Order): Invoice {
  let taxCents = 0;
  const lines = order.items.map(toLine);
  prepareCustomerCopy(order);
  taxCents = computeTax(lines, order.region);
  return assembleInvoice(lines, taxCents);
}
```

### Do: declare values at first use

```typescript
function buildInvoice(order: Order): Invoice {
  const lines = order.items.map(toLine);
  prepareCustomerCopy(order);
  const taxCents = computeTax(lines, order.region);
  return assembleInvoice(lines, taxCents);
}
```
