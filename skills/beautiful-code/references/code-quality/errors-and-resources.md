# Errors and Resources

Design failure paths with the successful path. Protect evidence, state, and resources when anything fails.

## Preserve failure meaning

- Preserve the original cause and add operation-specific context without exposing secrets.
- Catch only when the current layer can recover, translate, enrich, or perform required cleanup.
- Use one consistent representation for expected failures. Do not unpredictably mix exceptions, `null`, booleans, and sentinel values.
- Distinguish user-correctable, retryable, and terminal failures when callers respond differently.
- Never discard failures silently. Make deliberate best-effort behavior observable.
- Inspect partial writes and side effects before retrying. Require idempotency or compensation where repetition can duplicate damage.

## Fail fast at the right boundary

- Validate untrusted input at the boundary and report errors in the caller's vocabulary.
- State preconditions, postconditions, invariants, and failure modes explicitly.
- Fail near the source when an impossible state appears. Do not repair corruption by guessing and continue.

## Cleanup and ownership

- Make the code that acquires a file, stream, connection, lock, subscription, timer, or worker responsible for deterministic release.
- Prefer scoped or language-supported structured cleanup.
- Put cleanup in `finally` when an operation between acquisition and release can throw.
- Define ownership and release order explicitly before transferring a resource across a boundary.
- Preserve the primary failure if cleanup also fails; surface both when the platform supports it.
- Apply deadlines, quotas, and size limits so failed or hostile work cannot hold resources indefinitely.

## Expected errors and impossible states

- Assert programmer assumptions and conditions that should be impossible if the code is correct.
- Validate expected bad input instead of asserting it.
- Keep assertions free of side effects.
- Keep valuable invariant checks enabled unless measured production cost requires otherwise.

## Examples

### Don't: collapse failures and leak the handle

```typescript
async function exportInvoice(id: string, path: string): Promise<boolean> {
  try {
    const handle = await fs.open(path, "w");
    const invoice = (await fetch(`/invoices/${id}`).then((r) => r.json())) as Invoice;
    await handle.write(render(invoice));
    await handle.close();
    return true;
  } catch {
    return false;
  }
}
```

### Do: classify failure, preserve cause, and guarantee cleanup

```typescript
async function exportInvoice(id: string, path: string): Promise<void> {
  const response = await fetch(`/invoices/${id}`).catch((cause) => {
    throw new InvoiceFetchError(`Network failure loading invoice ${id}`, { cause });
  });
  if (response.status === 404) throw new InvoiceNotFoundError(id);
  if (!response.ok) throw new InvoiceFetchError(`Invoice ${id} failed: ${response.status}`);

  const invoice = invoiceSchema.parse(await response.json());
  const handle = await fs.open(path, "w");
  try {
    await handle.write(render(invoice));
  } finally {
    await handle.close();
  }
}
```
