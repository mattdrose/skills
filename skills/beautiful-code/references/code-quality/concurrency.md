# Concurrency

Make application concurrency correct by construction. State ownership, ordering, capacity, cancellation, and failure behavior explicitly.

## Independent work

- Run operations concurrently only when they have no data or ordering dependency.
- Preserve ordering only where correctness requires it; encode the dependency with awaits, messages, or synchronization rather than timing.

## Shared mutable state

- Minimize shared mutable state. Prefer immutable messages, partitioned state, or one owner per mutable value.
- Document the invariant protected by each lock and keep critical sections small.
- Never call unknown, reentrant, or blocking code while holding a lock.
- Require a clear happens-before argument for every shared-state claim. “Unlikely” is not synchronization.

## Atomic decisions

- Protect the complete decision and update. Never split check-then-act work across an `await`.
- Express coordination as one application operation with one owner rather than relying on timing.

### Don't: split a reservation across an `await`

```typescript
if (!(await seats.isAssigned(seatId))) {
  await seats.assign(seatId, userId);
}
```

### Do: give one owner an atomic operation

```typescript
const assigned = await seats.assignIfFree({ seatId, userId });
if (!assigned) throw new SeatAlreadyAssignedError(seatId);
```

## Cancellation, bounds, and failure

- Bound worker counts, queues, retries, and fan-out to prevent memory, connection, and service exhaustion.
- Propagate cancellation and deadlines through every blocking boundary.
- Await spawned work or attach explicit supervision. Surface background failures.
- Define shutdown: stop accepting work, propagate cancellation, and await or deliberately abandon tasks safely.
- Make retried or duplicate operations idempotent when delivery can repeat.
- For actors or workers, define message schemas, ownership, supervision, backpressure, ordering, duplication, and partial-failure behavior.
- Use shared work boards only when loose coordination justifies their complexity; define atomic claims, expiry, completion, and cleanup.
- Exercise contention, cancellation, failure, overload, and shutdown.
- Use barriers, controlled schedulers, or explicit synchronization points. Never use sleeps as proof of ordering.
- Test boundedness and verify that every task failure reaches an owner.

## Examples

### Don't: use unbounded fire-and-forget work and a timer

```typescript
async function importRecords(records: RawRecord[]): Promise<void> {
  for (const record of records) {
    void ingest(record);
  }
  await sleep(10_000);
}
```

### Do: bound, await, propagate cancellation, and surface failures

```typescript
async function importRecords({
  records,
  signal,
}: {
  records: RawRecord[];
  signal: AbortSignal;
}): Promise<void> {
  let nextIndex = 0;
  const failures: unknown[] = [];
  const workers = Array.from({ length: Math.min(4, records.length) }, async () => {
    while (true) {
      signal.throwIfAborted();
      const index = nextIndex++;
      if (index >= records.length) return;

      try {
        signal.throwIfAborted();
        await ingest(records[index], signal);
      } catch (error) {
        failures.push(error);
      }
    }
  });

  const results = await Promise.allSettled(workers);
  for (const result of results) {
    if (result.status === "rejected") failures.push(result.reason);
  }
  if (failures.length > 0) {
    throw new ImportBatchError(failures, records.length);
  }
}
```

### Don't: serialize independent application work

```typescript
const profile = await fetchProfile(userId);
const orders = await fetchOrders(userId);
const alerts = await fetchAlerts(userId);
```

### Do: express independence directly

```typescript
const [profile, orders, alerts] = await Promise.all([
  fetchProfile(userId),
  fetchOrders(userId),
  fetchAlerts(userId),
]);
```
