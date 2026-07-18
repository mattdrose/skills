# Core Principles

Clean code makes intent and consequences easy to discover.

- Optimize for the next reader and change, not the fewest keystrokes.
- Keep behavior obvious; surprising code demands investigation or explanation.
- Make one concept live in one authoritative place.
- Separate policy from mechanism and high-level intent from low-level detail.
- Keep dependencies explicit, narrow, and directed toward stable interfaces.
- Delete dead paths, stale compatibility layers, and speculative flexibility.
- Improve nearby code without turning a focused change into an unrelated rewrite.
- Judge cleanliness by change safety: readable code without reliable verification is still risky.

A review should distinguish defects from preferences. Ask whether a choice obscures intent, permits invalid states, duplicates knowledge, or makes likely changes unsafe.

## Examples

### Don't: let one rule live in two places

```typescript
// The free-shipping rule is duplicated and the two copies can drift apart
function cartBanner(cart: Cart): string {
  return cart.subtotalCents >= 5000 ? "Free shipping!" : "Almost there";
}

function shippingCents(cart: Cart): number {
  return cart.subtotalCents >= 5000 ? 0 : 799;
}
```

### Do: give the concept one authoritative home

```typescript
const FREE_SHIPPING_THRESHOLD_CENTS = 5000;

function qualifiesForFreeShipping(cart: Cart): boolean {
  return cart.subtotalCents >= FREE_SHIPPING_THRESHOLD_CENTS;
}

function cartBanner(cart: Cart): string {
  return qualifiesForFreeShipping(cart) ? "Free shipping!" : "Almost there";
}

function shippingCents(cart: Cart): number {
  return qualifiesForFreeShipping(cart) ? 0 : 799;
}
```

### Don't: keep speculative flexibility no caller uses

```typescript
// No caller passes options; every reader still has to reason about them
function exportReport(
  report: Report,
  options: { format?: "csv" | "xml" | "json"; legacyMode?: boolean } = {},
): string {
  if (options.legacyMode) return legacyExport(report);
  if (options.format === "xml") return toXml(report.rows);
  if (options.format === "json") return JSON.stringify(report.rows);
  return toCsv(report.rows);
}
```

### Do: support what exists today

```typescript
function exportReportCsv(report: Report): string {
  return toCsv(report.rows);
}
```
