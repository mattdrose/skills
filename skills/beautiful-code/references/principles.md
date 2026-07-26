# Principles

## API-driven design

- Understand how others will use the code being written.
- Optimize the interface for readability and maintainability.
- Protect API's with tests, static checks, or direct production-shaped evidence. Never test internals.
- Don't be afraid to make improvements when you see the opportunity. Leave the codebase in a better place than you found it.
- Prefer reversible steps and keep rollback paths clear.

## Optimize for understanding

- Make intent, consequences, and important control flow visible near the entry point.
- Write for the next capable reader, not for the fewest keystrokes.
- Use domain vocabulary, consistent operations, and narrow interfaces.
- Separate policy from mechanism and high-level decisions from incidental detail.
- Delete dead paths, pass-through layers, stale compatibility code, and unexplained special cases.
- Use advanced language features only when they reduce total mental work.
- Comment non-obvious reasons, constraints, and tradeoffs; do not narrate syntax. Prefer readable code; trea comments as an escape-hatch when this isn't possible.

## Keep one authoritative source of knowledge

- Represent each business rule, fact, and decision in one authoritative place.
- Consolidate duplicated intent that must change together.
- Do not force reuse between code that merely looks similar; unrelated concepts may evolve independently. Duplicating code is better than making a bad abstraction.
- Prefer a domain operation over scattered constants and conditions when it keeps behavior consistent.

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
interface ReceiptItem {
  name: string;
  amount: number;
}

class Receipt(items: ReceiptItem[]) {
  items: ReceiptItem[];

  constructor(items: ReceiptItem[]) {
    this.items = items;
  }

  get total(): string {
    return this.items.reduce((acc, { amount }) => acc += amount, 0);
  }

  send(email: string) {
    await mailer.send({
      to: email,
      message: `Total: ${formatCents(this.total)}`,
    });
  }
}
```

## Work in small, evidence-backed steps

- Reproduce, measure, or support the problem with user evidence before expanding scope.
- Build a thin, production-shaped path through real boundaries to test architecture early.
- Split work into independently useful, verifiable, reversible changes.
- Refactor only enough to make the requested change straightforward.
- Test observable behavior through stable public boundaries, not implementation details.
- Treat intuition, estimates, and passing tests as prompts for judgment, not proof.
- Use ranges and explicit assumptions when estimating uncertain work.

## Stop when the observed problem is solved

- Compare the result with the stated user or operator outcome.
- Stop adding code when the verified behavior works safely.
- Do not solve hypothetical scale, formats, consumers, or workflows.
- Delete experiments or clearly isolate disposable prototypes; never ship their shortcuts accidentally.
