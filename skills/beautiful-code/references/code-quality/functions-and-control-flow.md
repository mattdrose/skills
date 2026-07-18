# Functions and Control Flow

## One operation and one abstraction level

- Make each function express one coherent operation at one abstraction level.
- Keep the main path visible near the entry point; move incidental parsing, formatting, persistence, and transport details behind named operations.
- Split work when parts change for different reasons or form independently nameable operations.
- Treat length as a signal, not a verdict; optimize for cohesion, visible control flow, and ease of verification.
- Do not fragment straightforward code into tiny pass-through functions that force readers to trace indirection.
- Delete dead code, unnecessary configuration, special cases, and forwarding layers when they add no policy or meaning.

## Explicit operations and side effects

- Name operations precisely and expose a small interface that hides incidental detail.
- Replace selector booleans with distinct operations when they choose unrelated behaviors.
- Make preconditions, mutations, I/O, and failure behavior apparent at the call site.
- Prefer direct control flow over clever compression, hidden callbacks, or language features that increase total mental work.
- Express similar operations consistently, but deduplicate branches only when they encode the same rule; similar syntax can represent different knowledge.
- Assess simplicity for a capable newcomer. Familiarity alone does not make an interface simple.

## Guard clauses and conditionals

- Keep the successful path visible; reject invalid input and exceptional cases early when guard clauses reduce nesting.
- Name complex conditions when the name reveals a domain rule.
- Keep essential state changes explicit rather than hiding them inside an apparently functional pipeline.
- Avoid nested ternaries and compressed reductions when a direct loop exposes conditions and mutation more clearly.
- Preserve input validation at trust boundaries; simplifying control flow must not weaken security or admit invalid states.

## Arguments and return values

- Keep parameter lists small. Pass a cohesive domain concept when several values belong together, not an arbitrary bag used only to shorten a signature.
- Prefer return values to output parameters or hidden mutation.
- Avoid mutating arguments unless the API explicitly promises mutation.
- Make required inputs and failure modes visible in names, types, and call structure.
- Do not hide dependencies in globals when callers need to understand effects or substitute collaborators.

## Examples

### Don't: combine unrelated behavior behind a selector

```typescript
function renderReceipt(order: Order, forEmail: boolean): string {
  return forEmail ? emailTemplate(order) : pdfLayout(order);
}

renderReceipt(order, true);
```

### Do: expose the operation

```typescript
function renderEmailReceipt(order: Order): string {
  return emailTemplate(order);
}

renderEmailReceipt(order);
```

### Don't: bury the main path and silently skip malformed data

```typescript
async function notifyOverdueAccounts(): Promise<void> {
  const rows = await db.query("SELECT * FROM accounts WHERE due_at < now()");
  for (const row of rows) {
    const email = row.email_address?.trim().toLowerCase();
    if (!email || !/^[^@]+@[^@]+$/.test(email)) continue;
    const body = `Balance: $${(row.balance_cents / 100).toFixed(2)}`;
    await smtp.send({ to: email, subject: "Overdue balance", body });
  }
}
```

### Do: expose validation, policy, and transport as named operations

```typescript
async function notifyOverdueAccounts(): Promise<void> {
  const accounts = await findOverdueAccounts();

  for (const account of accounts) {
    const recipient = validRecipient(account);
    if (!recipient) continue;

    await sendOverdueNotice({ recipient, account });
  }
}
```

### Don't: hide mutation in a value-returning function

```typescript
function discountedTotalCents(order: Order): number {
  order.totalCents = Math.round(order.totalCents * 0.9);
  return order.totalCents;
}
```

### Do: return a value and leave the input unchanged

```typescript
function discountedTotalCents(order: Order): number {
  return Math.round(order.totalCents * 0.9);
}
```

### Don't: compress state changes into one expression

```typescript
const totals = orders.reduce(
  (result, order) => ({
    ...result,
    [order.status]: (result[order.status] ?? 0) + order.total,
  }),
  {} as Record<string, number>,
);
```

### Do: show the control flow directly

```typescript
const totals: Record<string, number> = {};
for (const order of orders) {
  totals[order.status] = (totals[order.status] ?? 0) + order.total;
}
```
