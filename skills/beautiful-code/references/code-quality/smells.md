# Code Smells

Treat a smell as a prompt to investigate, not proof that code is wrong. Trace a concrete consequence, identify the change pressure, and judge the code in context.

## Intent

**Misleading names**

- Smell: You must read the body or callers to learn what a name means, or the name contradicts what the code does.
- Fix: Replace misleading, inconsistent, vague, or unexplained names with domain language.

**Don't:**

```typescript
const data = users.filter((u) => u.isActive);
```

**Do:**

```typescript
const activeUsers = users.filter((u) => u.isActive);
```

**Magic values**

- Smell: Bare literals appear in logic with no name explaining what they represent or in what unit.
- Fix: Name magic values, units, and business rules.

**Don't:**

```typescript
if (elapsedMs > 2592000000) applyDiscount(user, 0.15);
```

**Do:**

```typescript
const THIRTY_DAYS_MS = 30 * 24 * 60 * 60 * 1000;
const LOYALTY_DISCOUNT_RATE = 0.15;

if (elapsedMs > THIRTY_DAYS_MS) applyDiscount(user, LOYALTY_DISCOUNT_RATE);
```

**Dense expressions**

- Smell: One expression or block mixes low-level mechanics with high-level policy, forcing readers to shift levels mid-read.
- Fix: Simplify dense expressions and keep each section at one level of abstraction.

**Don't:**

```typescript
const price =
  base * (1 + (region === "EU" ? 0.2 : 0.1)) - (hasCoupon ? base * 0.05 : 0);
```

**Do:**

```typescript
const taxRate = region === "EU" ? EU_TAX_RATE : DEFAULT_TAX_RATE;
const couponDiscount = hasCoupon ? base * COUPON_RATE : 0;
const price = base * (1 + taxRate) - couponDiscount;
```

**Compensating comments**

- Smell: A comment restates what the code does or explains something a better name would make obvious.
- Fix: Improve code instead of adding comments that compensate for unclear intent.

**Don't:**

```typescript
// check if the user can rent a car
if (user.age >= 25 && user.licenseYears >= 2) rent(user);
```

**Do:**

```typescript
const canRentCar = user.age >= 25 && user.licenseYears >= 2;
if (canRentCar) rent(user);
```

**Dead code**

- Smell: Code has no callers, guards a case that can no longer occur, or exists only "in case we need it".
- Fix: Delete dead code, stale fallbacks, and abstractions with no current use.

**Don't:**

```typescript
function exportLegacyReport() {
  // unused since 2023, kept "just in case"
}
```

**Do:**

```typescript
// Deleted. Version control remembers it if it is ever needed.
```

## Responsibility

**Long function**

- Smell: Describing the function honestly requires "and", or blank lines and comments partition it into named stages.
- Fix: Split functions that mix unrelated phases or abstraction levels; first consider simplifying the algorithm.

**Don't:**

```typescript
function processOrder(order: Order) {
  // validate
  // ...20 lines...
  // charge payment
  // ...20 lines...
  // send confirmation
  // ...20 lines...
}
```

**Do:**

```typescript
class Order {
  function process() {
    this.validate();
    this.charge();
    this.sendConfirmation();
  }
}
```

**Feature envy**

- Smell: A method reads several fields of another object to compute something that object could answer itself.
- Fix: Move behavior toward the data and rules it governs when another unit knows their internals better than their owner.

**Don't:**

```typescript
class Order {
  format(): string {
    const total = this.subtotalCents + this.subtotalCents * this.taxRate;
    return formatCents(total);
  }
}
```

**Do:**

```typescript
class Order {
  get total(): number {
    return this.subtotalCents + this.subtotalCents * this.taxRate;
  }

  format(): string {
    return formatCents(this.total);
  }
}
```

**Policy in infrastructure**

- Smell: Business rules sit beside queries, sockets, or framework glue and cannot be tested without them.
- Fix: Keep high-level policy out of low-level infrastructure.

**Don't:**

```typescript
// 2% transfer fee buried in the query
db.accounts.update({ balance: balance - amount * 1.02 });
```

**Do:**

```typescript
const amountWithFee = applyTransferFee(amount);
db.accounts.update({ balance: balance - amountWithFee });
```

**Data clumps**

- Smell: The same cluster of parameters or fields recurs across multiple signatures.
- Fix: Gather values that repeatedly travel together into a meaningful domain value.

**Don't:**

```typescript
function ship(street: string, city: string, postal: string, country: string) {}
function bill(street: string, city: string, postal: string, country: string) {}
```

**Do:**

```typescript
function ship(destination: Address) {}
function bill(destination: Address) {}
```

**Long parameter list**

- Smell: A signature takes many arguments, and callers pass positional values that are easy to swap or ignore.
- Fix: Replace long parameter lists when they reveal a missing concept or excessive coupling; do not hide unrelated inputs in a bag.

**Temporary field**

- Smell: Fields are null or meaningless except during certain operations.
- Fix: Model temporary fields or unclear lifecycle states as explicit phases or separate objects.

**Don't:**

```typescript
class Importer {
  currentRow?: Row; // only meaningful while import() runs
}
```

**Do:**

```typescript
class Importer {
  import(rows: Row[]) {
    for (const row of rows) this.importRow(row);
  }

  private importRow(row: Row) {}
}
```

**Repeated switches**

- Smell: The same switch or if-else ladder on a type tag appears in more than one place.
- Fix: Consolidate repeated branching by kind when adding a kind requires finding every switch. Use a strategy, lookup, or polymorphism only when the variation is stable enough to justify it.

**Don't:**

```typescript
// the same switch appears again in labelFor() and priceFor()
switch (method.kind) {
  case "standard":
    return 5;
  case "express":
    return 2;
  case "overnight":
    return 1;
}
```

**Do:**

```typescript
function deliveryDays(kind: ShippingMethod["kind"]): number {
  switch (method.kind) {
    case "standard":
      return 5;
    case "express":
      return 2;
    case "overnight":
      return 1;
  }
}
```

**Refused bequest**

- Smell: A subclass overrides methods to throw or no-op, or callers check the concrete type before use.
- Fix: Replace inheritance when a subtype cannot honor the parent's expectations; prefer composition.

**Don't:**

```typescript
class Penguin extends Bird {
  fly(): never {
    throw new Error("penguins cannot fly");
  }
}
```

**Do:**

```typescript
class Penguin implements Swimmer {
  swim() {}
}
```

## Coupling

**Message chains**

- Smell: Call sites chain through several objects (`a.getB().getC().value`) to reach data they need.
- Fix: Replace long object-navigation chains with intention-level operations. Do not make callers depend on an internal graph's shape.

**Don't:**

```typescript
const city = order.getCustomer().getAddress().getCity();
```

**Do:**

```typescript
const city = order.shippingCity();
```

**Hidden dependencies**

- Smell: A function's behavior depends on state that never appears in its signature.
- Fix: Pass dependencies explicitly instead of retrieving hidden globals or services.

**Don't:**

```typescript
function chargeSubscription(subscription: Subscription) {
  return ServiceLocator.get("payments").charge(subscription.priceCents);
}
```

**Do:**

```typescript
class StripeGateway extends Gateway {
  constructor(stripe: Stripe) {
    this.gateway = stripe;
  }

  function chargeSubscription(subscription: Subscription): Charge {
    return this.gateway.charge(subscription.priceCents);
  }
}
```

**Dependency cycles**

- Smell: Two modules import each other, or a low-level module imports the policy that uses it.
- Fix: Break dependency cycles and keep base modules unaware of concrete implementations.

**Don't:**

```typescript
// order.ts
import { sendOrderEmail } from "./email";

// email.ts
import { Order } from "./order";
```

**Do:**

```typescript
// email.ts depends on a plain shape, not the order module
export function sendOrderEmail({
  id,
  totalCents,
}: {
  id: string;
  totalCents: number;
}) {}
```

**Broad interfaces**

- Smell: Implementers stub methods they never use, or every consumer touches a small fraction of the interface.
- Fix: Narrow broad interfaces to what consumers need.

**Don't:**

```typescript
class MemoryCache implements Store {
  read(key: string) {}
  write(key: string, data: Data) {}
  compact() {} // stub
  replicate() {} // stub
}
```

**Do:**

```typescript
class MemoryCache implements Reader, Writer {
  read(key: string) {}
  write(key: string, data: Data) {}
}
```

**Vendor leakage**

- Smell: SDK or framework types appear in domain signatures far from the integration point.
- Fix: Prevent vendor types and infrastructure details from leaking into core policy.

**Don't:**

```typescript
function saveUser(user: User): Promise<DynamoDB.PutItemOutput> {}
```

**Do:**

```typescript
function saveUser(user: User): Promise<void> {}
```

**Inappropriate intimacy**

- Smell: A module reads or mutates another's private state, or breaks when the other's internal layout changes.
- Fix: Stop modules from reaching into each other's internals; narrow the interface or move ownership.

**Don't:**

```typescript
if (cart["_items"].length > 0) checkout(cart);
```

**Do:**

```typescript
if (!cart.isEmpty()) checkout(cart);
```

**Needless indirection**

- Smell: A layer only renames or forwards calls, and its removal would change no behavior and shorten the path.
- Fix: Remove indirection that contributes no policy, isolation, translation, or stability; keep useful boundaries.

**Don't:**

```typescript
function getUserById(id: string): User {
  return findUser(id); // renames, adds nothing
}
```

**Do:**

```typescript
findUser(id);
```

## Change risk

**Duplicated knowledge**

- Smell: Fixing one bug requires making the same edit in several places.
- Fix: Centralize duplicated knowledge so fixes cannot diverge; do not abstract merely similar syntax that may evolve independently.

**Don't:**

```typescript
function invoiceTotal(cents: number): number {
  return cents > 100_000 ? cents * 0.95 : cents;
}

function quoteTotal(cents: number): number {
  return cents > 100_000 ? cents * 0.95 : cents;
}
```

**Do:**

```typescript
function discountedTotal(cents: number): number {
  return cents > 100_000 ? cents * 0.95 : cents;
}
```

**Shotgun surgery**

- Smell: One conceptual change fans out into edits across many files.
- Fix: Move a scattered rule to one owning boundary when a single change requires edits across many units.

**Don't:**

```typescript
// checkout.ts, invoice.ts, and email.ts each format money themselves;
// changing the display means editing all three
const label = `$${(cents / 100).toFixed(2)}`;
```

**Do:**

```typescript
// money.ts — the one place currency display changes
export function formatMoney(cents: number): string {
  return `$${(cents / 100).toFixed(2)}`;
}
```

**Tangled responsibilities**

- Smell: Unrelated changes keep colliding in the same unit's history and reviews.
- Fix: Separate responsibilities when one unit changes for unrelated concerns.

**Don't:**

```typescript
class Checkout {
  calculateTotal() {}
  renderReceiptHtml() {}
  retryFailedWebhooks() {}
}
```

**Do:**

```typescript
class Checkout {
  total() {}
}

class Receipt {
  render() {}
}

class WebhookRetrier {
  retry() {}
}
```

**Flag arguments**

- Smell: Callers pass bare `true`/`false` literals whose meaning you must look up at the definition.
- Fix: Replace boolean selectors with intention-revealing operations or distinct behavior.

**Don't:**

```typescript
render(order, true);
```

**Do:**

```typescript
renderCompact(order, { compact = true });
```

**Primitive obsession**

- Smell: Raw strings or numbers carry domain meaning (ids, money, emails) and are re-validated everywhere or nowhere.
- Fix: Constrain primitive values when they permit invalid states; validate at trust boundaries.

**Don't:**

```typescript
function sendReceipt(email: string, amountCents: number) {}

sendReceipt("not-an-email", -500); // compiles fine
```

**Do:**

```typescript
function sendReceipt(email: EmailAddress, amount: Money) {}
```

**Unsafe multi-step writes**

- Smell: Consecutive writes to different stores or rows with nothing to roll back or compensate if one fails.
- Fix: Protect multi-step writes with transactions, recovery, or idempotency. Partial writes must not silently leave inconsistent state.

**Don't:**

```typescript
await orders.insert(order);
await inventory.decrement(order.items); // a crash here strands the order
```

**Do:**

```typescript
await db.transaction(async (tx) => {
  await tx.orders.insert(order);
  await tx.inventory.decrement(order.items);
});
```

**Speculative generality**

- Smell: Parameters, hooks, or interfaces with exactly one implementation and no concrete second use in sight.
- Fix: Remove speculative flexibility until a real use requires it.

**Don't:**

```typescript
interface ExportStrategy {
  export(data: Data): string;
}

class CsvExportStrategy implements ExportStrategy {} // the only one
```

**Do:**

```typescript
function exportCsv(data: Data): string {}
```

**Big-bang rewrites**

- Smell: A diff mixes behavior changes with restructuring, or touches far more than the stated problem.
- Fix: Preserve behavior while restructuring; broad rewrites amplify risk and obscure the cause of failures.

**Don't:**

```typescript
// One PR: swap the ORM, rename every module, and fix the rounding bug.
```

**Do:**

```typescript
// PR 1: fix the rounding bug (behavior change, tested).
// PR 2: rename modules (structure only, no behavior change).
```

## Verification

**Happy-path-only tests**

- Smell: Tests exercise only the happy path while error branches and edge inputs have none.
- Fix: Cover boundaries, failures, state transitions, side effects, and important regressions.

**Don't:**

```typescript
it("divides", () => {
  expect(divide(10, 2)).toBe(5);
});
```

**Do:**

```typescript
it("divides", () => {
  expect(divide(10, 2)).toBe(5);
});

it("rejects dividing by zero", () => {
  expect(() => divide(10, 0)).toThrow();
});
```

**Structure-coupled tests**

- Smell: Tests break on refactors that preserve behavior, or assert on mock call order instead of outcomes.
- Fix: Test observable behavior through stable boundaries, not private structure or call sequences.

**Don't:**

```typescript
expect(cache.get).toHaveBeenCalledWith("user:42");
expect(db.query).not.toHaveBeenCalled();
```

**Do:**

```typescript
expect(await getUser("42")).toEqual({ id: "42", name: "Ada" });
```

**Nondeterministic tests**

- Smell: Tests sleep, retry until green, or fail intermittently.
- Fix: Control time, randomness, concurrency, and external state; never synchronize with arbitrary sleeps.

**Don't:**

```typescript
await sleep(2000); // hope the job finished
expect(job.status).toBe("done");
```

**Do:**

```typescript
await job.completed;
expect(job.status).toBe("done");
```

**Swallowed errors**

- Smell: Empty catch blocks, ignored promise rejections, or logs that omit what failed and for which input.
- Fix: Surface swallowed errors and background failures with actionable context, without exposing secrets or sensitive data.

**Don't:**

```typescript
try {
  await syncAccount(account);
} catch {}
```

**Do:**

```typescript
try {
  await syncAccount(account);
} catch (error) {
  logger.error("account sync failed", { accountId: account.id, error });
  throw error;
}
```

**Unguarded invalid states**

- Smell: Objects can be built in states the domain forbids, or a failure leaves no trace anywhere.
- Fix: Treat invalid-state construction, partial persistence, and unobserved failure as correctness and security risks, not merely style concerns.

**Don't:**

```typescript
const order = new Order();
order.totalCents = -500; // nothing stops this
```

**Do:**

```typescript
class Order {
  constructor(readonly totalCents: number) {
    if (totalCents < 0) throw new Error("total cannot be negative");
  }
}
```

**Unverified refactors**

- Smell: A refactor's only safety claim is a narrow or pre-existing test run.
- Fix: Demand evidence that a proposed refactor preserves behavior. A passing narrow test does not establish system-wide safety.

**Don't:**

```typescript
// "Refactored pricing — the one existing pricing test still passes."
```

**Do:**

```typescript
// Ran the full suite plus a before/after comparison of computed
// prices over representative inputs; outputs are identical.
```

## Choosing a response

1. State the concrete maintenance, correctness, security, or failure-recovery problem.
2. Decide whether the signal is local or evidence of misplaced ownership.
3. Identify the next likely change and the pressure it places on the design.
4. Choose the smallest focused transformation that removes that pressure.
5. Verify behavior before and after the change.
6. Leave the code alone when the response would add more complexity than the smell costs.

Prefer deletion, renaming, moving responsibility, narrowing an interface, or consolidating one rule over a broad rewrite. Introduce a domain type, strategy, or new boundary only when it makes invalid states harder, ownership clearer, or the next likely change easier.
