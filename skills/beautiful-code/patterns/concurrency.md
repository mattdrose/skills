# Concurrency

Protect business invariants across overlapping transactions, not merely individual rows.

## Optimistic locking

Store a version and reject stale writes. Best when conflicts are uncommon and callers can retry or resolve them.

### Don't: read-modify-write with no conflict check

```typescript
async function addFunds(accountId: string, amountCents: number): Promise<void> {
  const account = await db.accounts.findById(accountId);
  account.balanceCents += amountCents;
  // a concurrent writer's update is silently overwritten
  await db.accounts.update(accountId, { balanceCents: account.balanceCents });
}
```

### Do: reject stale writes with a version

```typescript
async function addFunds({
  accountId,
  amountCents,
}: {
  accountId: string;
  amountCents: number;
}): Promise<void> {
  const account = await db.accounts.findById(accountId);
  const updated = await db.accounts.updateWhere(
    { id: accountId, version: account.version },
    {
      balanceCents: account.balanceCents + amountCents,
      version: account.version + 1,
    },
  );
  if (updated === 0) throw new ConcurrencyConflictError("account changed; retry");
}
```

## Pessimistic locking

Acquire database locks before changing contested data. Keep transactions short and define lock order to limit deadlocks.

## Coarse-grained locking

Lock an aggregate when its invariant spans several records. It is simpler but reduces concurrency.

### Don't: per-row checks for an invariant spanning rows

```typescript
// invariant: a policy's allocations must sum to 100%
async function setAllocation(allocId: string, percent: number): Promise<void> {
  await db.allocations.updateWithVersion(allocId, { percent });
  // two concurrent edits each pass their own row check; the policy total ends at 120%
}
```

### Do: lock the aggregate that owns the invariant

```typescript
async function setAllocation({
  policyId,
  allocId,
  percent,
}: {
  policyId: string;
  allocId: string;
  percent: number;
}): Promise<void> {
  const policy = await db.policies.findById(policyId);
  assertSumsTo100(await db.allocations.byPolicy(policyId), allocId, percent);

  const updated = await db.policies.updateWhere(
    { id: policyId, version: policy.version },
    { version: policy.version + 1 },
  );

  if (updated === 0) {
    throw new ConcurrencyConflictError("policy changed; retry");
  }

  await db.allocations.update(allocId, { percent });
}
```

## Implicit locking

Centralize lock acquisition in infrastructure when omissions are the main risk; keep the behavior observable.

## Review

Identify the invariant, conflict boundary, lock lifetime, retry semantics, and user-visible outcome. A transaction alone does not prevent lost updates.
