# Comments

Comments should preserve information the code cannot express clearly.

Useful comments explain:

- why a non-obvious constraint or tradeoff exists;
- a protocol, compatibility, security, or performance requirement;
- the reason an apparently simpler implementation is wrong;
- public API behavior that callers must know.

Remove or revise comments that:

- paraphrase the next line;
- describe behavior that has changed;
- contain history better kept in version control or an issue tracker;
- excuse confusing code that can be clarified directly;
- leave disabled code in place.

Treat TODOs as actionable only when they identify a specific remaining problem and, where
appropriate, link to tracked work. Verify every edited comment against the resulting behavior.

## Examples

### Don't: paraphrase the code

```typescript
// loop over the line items and add up the total
let total = 0;
for (const item of order.lineItems) {
  // add the item price times quantity
  total += item.priceCents * item.quantity;
}
```

### Do: record only what the code cannot say

```typescript
// Totals stay in integer cents
// floating-point dollars caused rounding bugs (#482)
function orderTotalCents(order: Order): number {
  return order.lineItems.reduce((sum, item) => {
    return sum + item.priceCents * item.quantity;
  }, 0);
}
```

### Don't: keep a comment describing behavior that changed

```typescript
// Retries three times with backoff before giving up
async function fetchProfile(id: string): Promise<Profile> {
  const res = await api.get(`/profiles/${id}`); // retries were removed; the comment now lies
  return parseProfile(res);
}
```

### Do: keep every comment true of the current code

```typescript
// Retries live in the api client; see fetchWithRetry (#519)
async function fetchProfile(id: string): Promise<Profile> {
  const res = await api.get(`/profiles/${id}`);
  return parseProfile(res);
}
```
