# Concurrency

Concurrent code must make ownership, ordering, and cancellation explicit.

- Minimize shared mutable state; prefer immutable messages or single-owner state.
- State which data is protected by each lock and keep critical sections small.
- Avoid calling unknown or blocking code while holding a lock.
- Define ordering requirements structurally rather than relying on timing.
- Propagate cancellation and deadlines through every blocking boundary.
- Bound queues, worker counts, retries, and fan-out to prevent resource exhaustion.
- Design repeated operations for idempotency when retries or duplicate delivery are possible.
- Test under contention and with controlled schedulers or synchronization points; sleeps are not
  synchronization.
- Ensure background failures are surfaced and shutdown waits for or safely abandons work.

A race that is unlikely is still a race. Require a clear happens-before argument for shared-state
correctness.

## Examples

### Don't: fire-and-forget with timing as synchronization

```typescript
let pending = 0;

async function importRecords(records: RawRecord[]): Promise<void> {
  for (const record of records) {
    pending += 1;
    void ingest(record).then(() => {
      pending -= 1;
    }); // failures vanish silently
  }
  await sleep(10_000); // hope the unbounded fan-out finished by now
}
```

### Do: bound the work, await it, surface failures

```typescript
async function importRecords({
  records,
  signal,
}: {
  records: RawRecord[];
  signal: AbortSignal;
}): Promise<void> {
  const limit = pLimit(4);
  const results = await Promise.allSettled(
    records.map((record) => {
      return limit(() => {
        return ingest(record, signal);
      });
    }),
  );

  const failures = results.filter((r) => r.status === "rejected");
  if (failures.length > 0) {
    throw new ImportBatchError(failures.length, records.length);
  }
}
```

### Don't: check-then-act across an await

```typescript
async function reserveSeat(showId: string, seat: string, userId: string): Promise<void> {
  const taken = await seats.isTaken(showId, seat);

  if (!taken) {
    // another caller can pass the same check before this write lands
    await seats.assign(showId, seat, userId);
  }
}
```

### Do: make the decision and the write one atomic operation

```typescript
async function reserveSeat({
  showId,
  seat,
  userId,
}: {
  showId: string;
  seat: string;
  userId: string;
}): Promise<void> {
  const reserved = await seats.assignIfFree({ showId, seat, userId });

  if (!reserved) {
    throw new SeatUnavailableError(showId, seat);
  }
}
```
