# Principles

## Preserve correctness

- Understand current behavior before changing structure.
- Preserve observable behavior outside the stated scope.
- Protect changes with tests, static checks, or direct production-shaped evidence.
- Include required validation, migrations, error handling, security, and observability; small does not mean incomplete.
- Separate behavior changes from structural cleanup when practical.
- Prefer reversible steps and keep rollback paths clear.

## Optimize for understanding

- Make intent, consequences, and important control flow visible near the entry point.
- Write for the next capable reader, not for the fewest keystrokes.
- Use domain vocabulary, consistent operations, and narrow interfaces.
- Separate policy from mechanism and high-level decisions from incidental detail.
- Delete dead paths, pass-through layers, stale compatibility code, and unexplained special cases.
- Use advanced language features only when they reduce total mental work.
- Comment non-obvious reasons, constraints, and tradeoffs; do not narrate syntax.

## Keep one authoritative source of knowledge

- Represent each business rule, fact, and decision in one authoritative place.
- Consolidate duplicated intent that must change together.
- Do not force reuse between code that merely looks similar; unrelated concepts may evolve independently.
- Prefer a domain operation over scattered constants and conditions when it keeps behavior consistent.

### Don't

```typescript
function shippingBanner(subtotalCents: number): string {
  return subtotalCents >= 5_000 ? "Free shipping" : "Shipping applies";
}

function shippingCents(subtotalCents: number): number {
  return subtotalCents >= 5_000 ? 0 : 799;
}
```

### Do

```typescript
const FREE_SHIPPING_THRESHOLD_CENTS = 5_000;

function qualifiesForFreeShipping(subtotalCents: number): boolean {
  return subtotalCents >= FREE_SHIPPING_THRESHOLD_CENTS;
}

function shippingBanner(subtotalCents: number): string {
  return qualifiesForFreeShipping(subtotalCents) ? "Free shipping" : "Shipping applies";
}

function shippingCents(subtotalCents: number): number {
  return qualifiesForFreeShipping(subtotalCents) ? 0 : 799;
}
```

## Prefer direct solutions

- Restate the user problem before accepting complexity inherited from the implementation.
- Reuse an existing product capability, platform feature, or suitable maintained dependency before building machinery.
- Choose the simplest adequate algorithm, then measure realistic workloads before optimizing.
- Support demonstrated behavior; reject speculative options, extension points, registries, and generic frameworks.
- Prefer direct, slightly longer code over compressed or indirect code.
- Remember that every option adds states, tests, failure modes, and maintenance cost.

### Don't

```typescript
function queryParameters(url: string): Record<string, string> {
  const result: Record<string, string> = {};
  for (const pair of (url.split("?")[1] ?? "").split("&")) {
    const [key, value] = pair.split("=");
    if (key) result[key] = decodeURIComponent(value ?? "");
  }
  return result;
}
```

### Do

```typescript
function queryParameters(url: string): Record<string, string> {
  return Object.fromEntries(new URL(url).searchParams);
}
```

## Make dependencies and change boundaries explicit

- Give components focused responsibilities and explicit, narrow dependencies.
- Keep unrelated concerns independently changeable; avoid global state, hidden dependencies, and internal-data chains.
- Isolate demonstrated volatility, external systems, and expensive-to-reverse choices behind the narrowest useful boundary.
- Do not introduce a boundary for an imagined variation; unused indirection is still complexity.
- Prevent vendor types, statuses, errors, and payloads from leaking through the application.
- Validate untrusted input and translate external failures at trust boundaries.

### Don't

```typescript
async function sendReceipt(orderId: string): Promise<void> {
  const row = await db.orders.rawRow(orderId);
  await mailer.send(row.cust_email_addr, `Total: ${row.amt_cents / 100}`);
}
```

### Do

```typescript
interface Receipt {
  email: string;
  totalFormatted: string;
}

async function sendReceipt(receipt: Receipt): Promise<void> {
  await mailer.send({ to: receipt.email, message: `Total: ${receipt.totalFormatted}` });
}
```

## Work in small, evidence-backed steps

- Reproduce, measure, or support the problem with user evidence before expanding scope.
- Build a thin, production-shaped path through real boundaries to test architecture early.
- Split work into independently useful, verifiable, reversible changes.
- Refactor only enough to make the requested change straightforward.
- Keep unrelated cleanup out of focused patches.
- Test observable behavior through stable public boundaries, not implementation details.
- Treat intuition, estimates, and passing tests as prompts for judgment, not proof.
- Use ranges and explicit assumptions when estimating uncertain work.

## Stop when the observed problem is solved

- Compare the result with the stated user or operator outcome.
- Stop adding code when the verified behavior works safely.
- Do not solve hypothetical scale, formats, consumers, or workflows.
- Leave nearby code better only when the improvement is necessary, low-risk, and evidence-backed.
- Delete experiments or clearly isolate disposable prototypes; never ship their shortcuts accidentally.
- Revisit the design only when new evidence exposes another problem.
