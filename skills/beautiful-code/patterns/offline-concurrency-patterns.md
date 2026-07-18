# Offline Concurrency Patterns

Use these for business transactions that outlive a database transaction; state the protected invariant first.


## Optimistic Offline Lock

**Problem:** prevent lost updates without holding a long lock. **Use when:** conflicts are uncommon and work can be retried or reconciled. **Tradeoff:** conflicts are detected late and may waste user work. **Review:** verify atomic version checks and an explicit conflict path; compare Pessimistic Lock when collisions are frequent or costly.

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

**Problem:** prevent conflicting work before it begins. **Use when:** collisions are likely or late rejection is unacceptable. **Tradeoff:** reduced concurrency, deadlocks, and abandoned locks require operational handling. **Review:** verify lease expiry, ownership, ordering, and recovery; prefer Optimistic Lock when retries are cheap.

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

**Problem:** protect an invariant spanning several objects. **Use when:** they must change as one consistency unit. **Tradeoff:** fewer locks simplify correctness but reduce concurrency and increase contention. **Review:** ensure the locked group matches the invariant and measure hotspot behavior.

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
  const order = await db.orders.findById(orderId);
  const updated = await db.orders.updateWhere(
    { id: orderId, version: order.version },
    { version: order.version + 1 },
  );

  if (updated === 0) throw new StaleEditError(orderId);
  await db.orderLines.update(line.id, { quantity: line.quantity });
}
```


## Implicit Lock

**Problem:** ensure required locking is not forgotten by callers. **Use when:** infrastructure can infer a uniform locking rule reliably. **Tradeoff:** hidden acquisition obscures blocking, ordering, and performance. **Review:** make lock scope observable and test every mutation path; use explicit locking for exceptions or nonuniform rules.
