# Names

Names should reveal purpose, units, and distinctions without requiring implementation lookup.

- Name values for what they mean, not their type or temporary role.
- Use domain vocabulary consistently; one concept should not acquire synonyms by accident.
- Distinguish similar values precisely: `scheduledAt` and `startedAt`, not `date1` and `date2`.
- Include units or representation when ambiguity is dangerous: `timeoutMs`, `priceCents`.
- Prefer searchable words over unexplained abbreviations and single letters outside tiny scopes.
- Name booleans and predicates so conditions read naturally.
- Name operations for observable behavior. A mutating method should not sound like a query.
- Avoid filler such as `data`, `info`, `manager`, `util`, or `process` unless it adds real meaning.

Rename when the current name causes a reader to form the wrong expectation, not merely because
another synonym sounds nicer.

## Examples

### Don't: force readers to look up what values mean

```typescript
function check(d1: number, d2: number, list: Sub[]): Sub[] {
  return list.filter((s) => s.date > d1 && s.date < d2 && s.flag);
}
```

### Do: reveal purpose, units, and distinctions

```typescript
function activeSubscriptionsRenewedBetween({
  periodStartMs,
  periodEndMs,
  subscriptions,
}: {
  periodStartMs: number;
  periodEndMs: number;
  subscriptions: Subscription[];
}): Subscription[] {
  return subscriptions.filter((sub) => {
    return sub.renewedAtMs > periodStartMs && sub.renewedAtMs < periodEndMs && sub.isActive;
  });
}
```

### Don't: name a mutating operation like a query

```typescript
// "get" promises a read, but this silently creates and saves a record
async function getAccount(email: string): Promise<Account> {
  let account = await accounts.findByEmail(email);
  if (!account) {
    account = await accounts.create(email);
  }
  return account;
}
```

### Do: name the operation for its observable behavior

```typescript
async function findOrCreateAccount(email: string): Promise<Account> {
  const existing = await accounts.findByEmail(email);
  return existing ?? (await accounts.create(email));
}
```
