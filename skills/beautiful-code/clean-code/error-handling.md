# Error Handling

Failure paths are part of the design, not cleanup around the main logic.

- Validate untrusted input at the boundary and return errors in the caller's vocabulary.
- Preserve the original cause and enough context to diagnose the operation that failed.
- Catch only errors the current layer can recover from, translate, or enrich.
- Never silently discard a failure. Deliberate best-effort behavior should be explicit and
  observable.
- Use one consistent strategy for expected failures; avoid mixing sentinel values, nulls, and
  exceptions unpredictably.
- Release resources with language-supported structured cleanup.
- Keep error handling from obscuring the successful path, but do not hide important recovery logic
  behind generic helpers.
- Distinguish retryable, user-correctable, and terminal failures where callers act differently.

Check partial-failure behavior carefully whenever work mutates state or crosses process boundaries.

## Examples

### Don't: collapse every failure into null

```typescript
async function loadInvoice(id: string): Promise<Invoice | null> {
  try {
    const res = await fetch(`/api/invoices/${id}`);
    return await res.json();
  } catch {
    // network failure, bad JSON, and "not found" are now identical
    return null;
  }
}
```

### Do: preserve cause and distinguish failures callers act on

```typescript
async function loadInvoice(id: string): Promise<Invoice> {
  let res: Response;

  try {
    res = await fetch(`/api/invoices/${id}`);
  } catch (cause) {
    throw new InvoiceFetchError(`Network failure loading invoice ${id}`, {
      cause,
    });
  }

  if (res.status === 404) {
    throw new InvoiceNotFoundError(id);
  }
  if (!res.ok) {
    throw new InvoiceFetchError(`Invoice ${id} request failed: ${res.status}`);
  }

  return parseInvoice(await res.json());
}
```

### Don't: leak resources when the operation throws

```typescript
async function exportOrders(path: string): Promise<void> {
  const handle = await fs.open(path, "w");
  const orders = await loadOrders(); // a throw here leaks the file handle
  await handle.write(serialize(orders));
  await handle.close();
}
```

### Do: release resources with structured cleanup

```typescript
async function exportOrders(path: string): Promise<void> {
  const handle = await fs.open(path, "w");

  try {
    await handle.write(serialize(await loadOrders()));
  } finally {
    await handle.close();
  }
}
```
