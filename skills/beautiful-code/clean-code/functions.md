# Functions

A function should express one coherent operation at one level of abstraction.

- Keep the main path visible; move incidental parsing, formatting, and transport details out of it.
- Split functions when their parts change for different reasons or can be named as separate
  operations.
- Prefer a returned value to an output parameter or hidden mutation.
- Avoid selector booleans that combine unrelated behaviors; expose distinct operations instead.
- Keep parameter lists small. When several values form one concept, pass that concept—not an
  arbitrary bag.
- Make preconditions, side effects, and failure behavior apparent at the call site.
- Replace repeated branches only when they represent the same rule; similar syntax can encode
  different knowledge.
- Do not fragment straightforward code into tiny indirections that make reading harder.

Length is a signal, not a verdict. Cohesion, abstraction level, and ease of verification matter
more.

## Examples

### Don't: combine unrelated behaviors behind a selector boolean

```typescript
function renderReceipt(order: Order, forEmail: boolean): string {
  if (forEmail) {
    return emailTemplate(order);
  }
  return pdfLayout(order);
}

renderReceipt(order, true); // call site reveals nothing about what happens
```

### Do: expose distinct operations

```typescript
function renderEmailReceipt(order: Order): string {
  return emailTemplate(order);
}

function renderPdfReceipt(order: Order): string {
  return pdfLayout(order);
}

renderEmailReceipt(order);
```

### Don't: bury the main path under incidental detail

```typescript
async function notifyOverdueAccounts(): Promise<void> {
  const rows = await db.query("SELECT * FROM accounts WHERE due_at < now()");
  for (const row of rows) {
    const email = row.email_address?.trim().toLowerCase();
    if (!email || !/^[^@]+@[^@]+$/.test(email)) continue;
    const balance = (row.balance_cents / 100).toFixed(2);
    const body = `Dear ${row.first_name}, your balance of $${balance} is overdue.`;
    await smtp.send({ to: email, subject: "Overdue balance", body });
  }
}
```

### Do: keep one operation at one level of abstraction

```typescript
async function notifyOverdueAccounts(): Promise<void> {
  const accounts = await findOverdueAccounts();

  for (const account of accounts) {
    const recipient = validRecipient(account);

    if (recipient) {
      await sendOverdueNotice(recipient, account);
    }
  }
}
```

### Don't: mutate an argument as a hidden side effect

```typescript
function applyDiscount(order: Order): number {
  order.totalCents = Math.round(order.totalCents * 0.9); // caller's order changed silently
  return order.totalCents;
}
```

### Do: return the result and leave inputs alone

```typescript
function discountedTotalCents(order: Order): number {
  return Math.round(order.totalCents * 0.9);
}
```
