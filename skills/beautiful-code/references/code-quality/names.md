# Names

- Reveal domain purpose, not type, storage, or temporary role.
- Use the vocabulary shared by users and maintainers; use one term for one concept.
- Distinguish similar values precisely: `scheduledAtMs` and `startedAtMs`, not `date1` and `date2`.
- Include units, currency, timezone, encoding, or representation when confusion could change behavior: `timeoutMs`, `priceCents`, `emailHtml`.
- Name booleans and predicates so conditions read as claims: `isActive`, `canRefund`, `hasExpired`.
- Name operations for observable behavior and side effects. Do not make creation, mutation, I/O, or deletion sound like a query.
- Prefer searchable words over unexplained abbreviations, clever wordplay, and single letters outside tiny scopes.
- Avoid vague containers such as `data`, `info`, `item`, `object`, `manager`, `service`, `util`, `helper`, and `process` when a domain term exists.
- Avoid redundant type words and encodings that can become false: prefer `accounts` to `accountList` and `phone` to `phoneString`.
- Keep names proportional to scope. Use short conventional names only where their meaning is immediate.
- Rename as understanding improves when the current name creates a wrong expectation.
- Do not rename merely because another synonym matches personal taste; preserve established domain language and public compatibility unless the benefit exceeds migration cost.
- Replace explanatory comments with accurate names when the comment only decodes an identifier. Keep comments that explain reasons or constraints.

## Review prompts

- Can a reader predict the value or behavior without opening the implementation?
- Does the name identify the domain concept, role, and relevant scope?
- Could units, sign, timezone, currency, or representation be mistaken?
- Do two names describe the same concept with accidental synonyms?
- Do similar names hide materially different states or timestamps?
- Does a boolean read naturally at its use site, including negation?
- Does a function name disclose mutation, persistence, network access, creation, or partial failure?
- Is a broad name hiding multiple responsibilities?
- Is an abbreviation established in the domain and consistently understood?
- Does the name remain truthful after the current change?
- Would renaming prevent a concrete defect or unsafe edit, or only express preference?
- If the name is public or persisted, does the clarity gain justify compatibility and migration cost?

## Examples

### Don't: hide purpose, units, and distinctions

```typescript
function check(d1: number, d2: number, list: Sub[]): Sub[] {
  return list.filter((s) => s.date > d1 && s.date < d2 && s.flag);
}
```

### Do: name domain meaning and representation

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
  return subscriptions.filter((subscription) => {
    return (
      subscription.renewedAtMs > periodStartMs &&
      subscription.renewedAtMs < periodEndMs &&
      subscription.isActive
    );
  });
}
```

### Don't: make a write sound like a read

```typescript
async function getAccount(email: string): Promise<Account> {
  const account = await accounts.findByEmail(email);
  return account ?? (await accounts.create(email));
}
```

### Do: disclose creation in the operation name

```typescript
async function findOrCreateAccount(email: string): Promise<Account> {
  const existingAccount = await accounts.findByEmail(email);
  return existingAccount ?? (await accounts.create(email));
}
```

### Don't: omit units where confusion changes behavior

```typescript
async function waitForJob(jobId: string, timeout: number): Promise<Job> {
  return jobs.wait(jobId, timeout);
}

await waitForJob(job.id, 30); // seconds or milliseconds?
```

### Do: encode the unit at the boundary

```typescript
async function waitForJob({
  jobId,
  timeoutMs,
}: {
  jobId: string;
  timeoutMs: number;
}): Promise<Job> {
  return jobs.wait(jobId, timeoutMs);
}

await waitForJob({ jobId: job.id, timeoutMs: 30_000 });
```

### Don't: use a broad name that conceals authority

```typescript
async function process(user: User, data: string): Promise<void> {
  await accounts.delete(data, user.id);
}
```

### Do: name the protected action and identifiers

```typescript
async function deleteAccount({
  actor,
  accountId,
}: {
  actor: User;
  accountId: string;
}): Promise<void> {
  await accounts.deleteAuthorized({ actorId: actor.id, accountId });
}
```
