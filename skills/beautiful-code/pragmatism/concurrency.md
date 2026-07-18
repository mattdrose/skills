# Concurrency

## Temporal decoupling

**Review question:** Which operations truly require a particular order or the caller to wait?

Separate dependencies in time from dependencies in data. Run independent work concurrently and use queues or schedulers when immediate completion is unnecessary. Preserve ordering only where correctness requires it.

### Don't: serialize work that has no data dependency

```typescript
async function loadDashboard(userId: string) {
  const profile = await fetchProfile(userId);
  const orders = await fetchOrders(userId); // waits for profile despite not needing it
  const alerts = await fetchAlerts(userId);
  return { profile, orders, alerts };
}
```

### Do: run independent operations concurrently

```typescript
async function loadDashboard(userId: string) {
  const [profile, orders, alerts] = await Promise.all([
    fetchProfile(userId),
    fetchOrders(userId),
    fetchAlerts(userId),
  ]);
  return { profile, orders, alerts };
}
```

## Shared state

**Review question:** Can two execution contexts read and modify the same state without a single consistency boundary?

Shared mutable state makes correctness depend on timing. Prefer immutable values, partitioned ownership, transactions, or synchronization around the full read-modify-write operation. Document the invariant protected by each lock or transaction.

### Don't: split a read-modify-write across the consistency boundary

```typescript
async function reserveSeat(showId: string, seat: string) {
  const show = await db.shows.get(showId);
  if (!show.reserved.includes(seat)) {
    // another request can reserve the same seat between the read and this write
    await db.shows.update(showId, { reserved: [...show.reserved, seat] });
  }
}
```

### Do: make the whole operation atomic

```typescript
async function reserveSeat({ showId, seat }: { showId: string; seat: string }) {
  const updated = await db.shows.updateOne(
    { id: showId, reserved: { $ne: seat } },
    { $push: { reserved: seat } },
  );
  if (updated.count === 0) throw new SeatTakenError(showId, seat);
}
```

## Actors and processes

**Review question:** Would isolated state and message passing simplify coordination?

Actors and independent processes own their state and communicate through messages. Define message schemas, supervision, backpressure, and failure behavior. Isolation reduces races but does not remove distributed-system concerns such as duplication and partial failure.

## Blackboard coordination

**Review question:** Do independent workers need to contribute to a result without knowing one another?

A shared work board can coordinate producers and consumers through data rather than direct calls. Define atomic claims, expiration, idempotency, completion, and cleanup. Use this model only when loose, opportunistic coordination is worth its operational complexity.
