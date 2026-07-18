# Concurrency Control

## Choosing lock scope and strategy

State the business invariant and consistency boundary before selecting a lock. Offline transactions outlive a database transaction; a transaction alone does not prevent lost updates. Prefer optimistic control when collisions are rare and retries are acceptable, pessimistic control when collision or late rejection is costly, and an aggregate-sized lock when an invariant spans records.

## Optimistic Offline Lock

**Problem:** Prevent lost updates without a long-held lock. **Use when:** conflicts are uncommon and work can be retried or reconciled. **Trade-off:** detection is late and may waste user work. **Review:** perform the version comparison and update atomically, surface an explicit merge/retry outcome, and never silently overwrite a stale edit.

### Don't: save an edited record without a version check

```typescript
async function saveTicket(edited: TicketEdit): Promise<void> {
  // overwrites whatever another agent saved while this form was open
  await db.tickets.update(edited.id, {
    status: edited.status,
    notes: edited.notes,
  });
}
```

### Do: verify the version atomically and surface conflicts

```typescript
async function saveTicket(edited: TicketEdit): Promise<void> {
  const updated = await db.tickets.updateWhere(
    { id: edited.id, version: edited.version },
    { status: edited.status, notes: edited.notes, version: edited.version + 1 },
  );
  if (updated === 0) throw new StaleEditError(edited.id); // caller shows a merge/retry UI
}
```

## Pessimistic Offline Lock

**Problem:** Prevent conflicting work before it starts. **Use when:** collisions are likely or late rejection is unacceptable. **Trade-off:** concurrency falls; deadlocks and abandoned locks need handling. **Review:** use ownership and expiring leases, define lock order and recovery, and keep held work short. A lock without expiry can strand data indefinitely.

### Don't: a checkout lock that can never be released

```typescript
async function beginEditing(docId: string, userId: string): Promise<void> {
  const acquired = await db.locks.tryInsert({ docId, userId });
  // if this user closes the tab, the document stays locked forever
  if (!acquired) throw new DocumentLockedError(docId);
}
```

### Do: a lease with expiry and ownership

```typescript
async function beginEditing({ docId, userId }: { docId: string; userId: string }): Promise<void> {
  const lease = await db.locks.acquire({
    docId,
    userId,
    expiresAt: Date.now() + 5 * 60_000, // heartbeat renews; abandoned locks lapse
  });

  if (!lease) throw new DocumentLockedError(docId);
}
```

## Coarse-Grained Lock

**Problem:** Protect an invariant spanning several objects. **Use when:** they form one consistency unit. **Trade-off:** fewer locks simplify correctness but increase contention and reduce concurrency. **Review:** lock or version the aggregate that owns the invariant, not independent rows; ensure transactional writes and measure hotspots.

### Don't: per-line versions for an invariant spanning the order

```typescript
// invariant: an order's total must equal the sum of its lines
async function updateLine(line: LineEdit): Promise<void> {
  await db.orderLines.updateWhere(
    { id: line.id, version: line.version },
    { quantity: line.quantity, version: line.version + 1 },
  );
  // two agents edit different lines; both row checks pass, the order invariant breaks
}
```

### Do: one version on the aggregate root

```typescript
async function updateLine({ orderId, line }: { orderId: string; line: LineEdit }): Promise<void> {
  await db.transaction(async (tx) => {
    const order = await tx.orders.findById(orderId);
    const updated = await tx.orders.updateWhere(
      { id: orderId, version: order.version },
      { version: order.version + 1 },
    );

    if (updated === 0) throw new StaleEditError(orderId);

    const lineUpdated = await tx.orderLines.updateWhere(
      { id: line.id, orderId },
      { quantity: line.quantity },
    );
    if (lineUpdated === 0) {
      throw new OrderLineNotFoundError({ orderId, lineId: line.id });
    }
  });
}
```

## Implicit Lock

**Problem:** Ensure callers cannot forget required locking. **Use when:** infrastructure can reliably infer one uniform rule. **Trade-off:** hidden acquisition obscures blocking, order, and performance. **Review:** make lock scope observable and test every mutation path; use explicit locking for exceptions and nonuniform rules.

## Review

Identify the invariant, conflict boundary, lock lifetime, acquisition order, transaction boundary, retry safety, and user-visible conflict outcome. Exercise concurrent reads and writes, stale forms, process death, lease expiry, deadlock recovery, and retries. Ensure retries cannot duplicate external effects and authorization is rechecked when delayed work finally commits.
