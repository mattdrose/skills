# Formatting

Formatting should expose structure and reduce navigation cost.

- Let the project formatter settle mechanical style.
- Put important exports and orchestration before supporting details when language conventions allow.
- Keep related declarations close and separate unrelated ideas with whitespace.
- Declare values near first use and keep their scopes narrow.
- Use consistent ordering across similar files so readers know where to look.
- Break dense expressions when intermediate names reveal meaning.
- Avoid alignment and hand formatting that create noisy diffs.
- Keep files cohesive; split them when distinct responsibilities can be named and changed
  independently.

Do not use formatting debates as substitutes for reviewing control flow, coupling, and correctness.

## Examples

### Don't: pack a dense expression into one line

```typescript
const eligible = users
  .filter(
    (u) => u.plan !== "free" && Date.now() - u.lastSeenMs < 30 * 24 * 60 * 60 * 1000 && !u.optedOut,
  )
  .map((u) => u.email);
```

### Do: break it apart with names that reveal meaning

```typescript
const THIRTY_DAYS_MS = 30 * 24 * 60 * 60 * 1000;

function isRecentlyActive(user: User): boolean {
  return Date.now() - user.lastSeenMs < THIRTY_DAYS_MS;
}

function acceptsEmail(user: User): boolean {
  return user.plan !== "free" && !user.optedOut;
}

const eligibleEmails = users
  .filter((user) => {
    return isRecentlyActive(user) && acceptsEmail(user);
  })
  .map((user) => user.email);
```

### Don't: declare everything up front, far from use

```typescript
function buildInvoice(order: Order): Invoice {
  let taxCents = 0; // declared here, meaningful thirty lines later
  let discount = null;
  const lines = order.items.map(toLine);

  // ... many lines of unrelated work ...
  discount = findDiscount(order.customer);
  taxCents = computeTax(lines, order.region);
  return assemble(lines, discount, taxCents);
}
```

### Do: declare values where they first mean something

```typescript
function buildInvoice(order: Order): Invoice {
  const lines = order.items.map(toLine);

  // ... many lines of unrelated work ...
  const discount = findDiscount(order.customer);
  const taxCents = computeTax(lines, order.region);
  return assemble(lines, discount, taxCents);
}
```
