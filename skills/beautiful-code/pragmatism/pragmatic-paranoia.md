# Defensive Engineering

## Contracts

**Review question:** Are valid inputs, outputs, state transitions, and failure modes explicit?

Define what a component requires and guarantees. Enforce contracts at trust boundaries and important invariants inside the system. A caller must satisfy preconditions; a callee must deliver its postconditions.

## Fail fast

**Review question:** Does invalid state stop near its source with useful diagnostic context?

Do not continue after an impossible condition and produce misleading downstream damage. Reject corruption early, preserve the evidence needed to diagnose it, and recover only when recovery semantics are genuinely defined.

### Don't: paper over impossible state and continue

```typescript
function settleInvoice(invoice: Invoice, payment: Payment) {
  if (payment.amountCents > invoice.balanceCents) {
    payment.amountCents = invoice.balanceCents; // silently mutates the evidence and carries on
  }
  invoice.balanceCents -= payment.amountCents;
}
```

### Do: fail fast with the evidence intact

```typescript
function settleInvoice({ invoice, payment }: { invoice: Invoice; payment: Payment }) {
  if (payment.amountCents > invoice.balanceCents) {
    throw new OverpaymentError(
      `payment ${payment.id} of ${payment.amountCents} exceeds balance ` +
        `${invoice.balanceCents} on invoice ${invoice.id}`,
    );
  }
  invoice.balanceCents -= payment.amountCents;
}
```

## Assertions

**Review question:** Are programmer assumptions executable and checked?

Assert conditions that should be impossible if the code is correct. Do not use assertions for expected input errors or side effects. Keep valuable checks enabled unless measurements prove their production cost is unacceptable.

### Don't: use assertions for expected input errors

```typescript
function registerUser(req: Request) {
  assert(isValidEmail(req.body.email)); // a user typo is expected input, not a programmer bug
  return users.create(req.body);
}
```

### Do: reject bad input at the boundary, assert the impossible

```typescript
function registerUser(req: Request) {
  if (!isValidEmail(req.body.email)) throw new BadRequestError("invalid email");
  const user = users.create(req.body);
  assert(user.id !== undefined, "create must assign an id");
  return user;
}
```

## Resource ownership

**Review question:** Is acquisition paired with deterministic release by the same owner?

The code that acquires a file, lock, connection, subscription, or allocation should make cleanup inevitable. Prefer scoped language constructs. Define ownership and release order explicitly when resources cross boundaries.

### Don't: leave resource release to the happy path

```typescript
async function exportAudit(rows: AuditRow[]) {
  const handle = await fs.open("/tmp/audit.csv", "w");
  for (const row of rows) {
    await handle.write(toCsvLine(row)); // a throw here leaks the file handle
  }
  await handle.close();
}
```

### Do: make cleanup inevitable at the acquisition site

```typescript
async function exportAudit(rows: AuditRow[]) {
  const handle = await fs.open("/tmp/audit.csv", "w");
  try {
    for (const row of rows) {
      await handle.write(toCsvLine(row));
    }
  } finally {
    await handle.close();
  }
}
```

## Bounded steps

**Review question:** Does the implementation rely only on what it can know and validate now?

Take small, observable steps instead of predicting distant conditions. Limit blast radius with timeouts, quotas, idempotency, checkpoints, and incremental delivery. Gather feedback before committing to the next horizon.
