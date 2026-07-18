# Supporting Patterns

These small patterns support larger designs; retain one only when its stated boundary or semantics are visible.


## Gateway

**Problem:** isolate an external system's protocol and vendor API. **Use when:** application code needs a stable, application-oriented boundary. **Tradeoff:** a leaky or overly generic wrapper adds indirection without insulation. **Review:** ensure protocol details, failures, and translation stop at the gateway.

### Don't: vendor API details spread through callers

```typescript
async function chargeOrder(order: Order): Promise<void> {
  const res = await stripe.paymentIntents.create({
    amount: order.totalCents,
    currency: "usd",
    metadata: { order_id: order.id },
  });
  if (res.status === "requires_action") retryQueue.push(order.id); // vendor states leak upward
}
```

### Do: a Gateway owning the protocol

```typescript
interface PaymentGateway {
  charge({ orderId, amount }: { orderId: string; amount: Money }): Promise<ChargeResult>;
}

class StripePaymentGateway implements PaymentGateway {
  async charge({ orderId, amount }: { orderId: string; amount: Money }): Promise<ChargeResult> {
    const res = await stripe.paymentIntents.create(toStripeParams({ orderId, amount }));
    return toChargeResult(res); // vendor statuses translated here, nowhere else
  }
}
```


## Mapper

**Problem:** translate between independent representations. **Use when:** neither side should depend on the other. **Tradeoff:** duplicated fields and lossy transformations drift silently. **Review:** inspect round-trip semantics, defaults, and error handling; do not add a mapper when representations are intentionally identical and coupled.


## Layer Supertype

**Problem:** share behavior required by every type in a layer. **Use when:** a genuine layer-wide contract or implementation exists. **Tradeoff:** empty base classes and broad inheritance create coupling. **Review:** require meaningful shared behavior; delete it if it serves only naming or convention.


## Separated Interface

**Problem:** reverse a package dependency while hiding an implementation. **Use when:** policy must own the abstraction implemented by infrastructure. **Tradeoff:** extra interfaces fragment navigation and can merely duplicate one class. **Review:** verify dependency direction changes and the interface speaks the consumer's vocabulary.


## Registry

**Problem:** make shared process-level objects globally reachable. **Use when:** explicit passing is genuinely impractical for stable infrastructure. **Tradeoff:** hidden dependencies, mutable global state, and test interference follow. **Review:** look for scoped lifetime and immutable registration; prefer explicit injection for ordinary dependencies.

### Don't: ordinary dependencies fetched from a global registry

```typescript
class InvoiceService {
  async send(invoiceId: string): Promise<void> {
    const mailer = Registry.get("mailer"); // hidden dependency, mutable global, test interference
    const invoices = Registry.get("invoiceRepo");
    await mailer.send(await invoices.byId(invoiceId));
  }
}
```

### Do: explicit injection with visible dependencies

```typescript
class InvoiceService {
  private readonly mailer: Mailer;
  private readonly invoices: InvoiceRepository;

  constructor({ mailer, invoices }: { mailer: Mailer; invoices: InvoiceRepository }) {
    this.mailer = mailer;
    this.invoices = invoices;
  }

  async send(invoiceId: string): Promise<void> {
    await this.mailer.send(await this.invoices.byId(invoiceId));
  }
}
```


## Value Object

**Problem:** model a concept defined by attributes rather than identity. **Use when:** equality and operations depend only on the value. **Tradeoff:** mutability breaks equality and persistence assumptions. **Review:** require value equality, validated construction, and practical immutability; use an entity when lifecycle identity matters.


## Money

**Problem:** represent monetary values without losing currency or arithmetic rules. **Use when:** amounts are compared, rounded, allocated, or converted. **Tradeoff:** naive numeric storage and implicit conversion produce financial errors. **Review:** verify currency-aware equality, decimal precision, rounding, and explicit exchange rates; never use a bare float.

### Don't: monetary math on bare floats

```typescript
function applyDiscount(invoice: { total: number }, percent: number): number {
  return invoice.total * (1 - percent / 100); // float drift, no currency, silent rounding
}

const owed = applyDiscount({ total: 19.99 }, 15) + 4.95;
```

### Do: a Money value object

```typescript
class Money {
  readonly cents: bigint;
  readonly currency: string;

  constructor({ cents, currency }: { cents: bigint; currency: string }) {
    this.cents = cents;
    this.currency = currency;
  }
  add(other: Money): Money {
    if (other.currency !== this.currency) throw new Error("currency mismatch");
    return new Money({
      cents: this.cents + other.cents,
      currency: this.currency,
    });
  }
  equals(other: Money): boolean {
    return this.cents === other.cents && this.currency === other.currency;
  }
}
```


## Special Case

**Problem:** remove repeated handling of a common exceptional value. **Use when:** absence or an exceptional state has stable, safe behavior. **Tradeoff:** the object can hide missing-data errors or invent misleading defaults. **Review:** ensure behavior is semantically valid and the case remains distinguishable when required; prefer explicit errors for unexpected absence.

### Don't: repeated null handling for a missing customer

```typescript
const customer = await customers.find(order.customerId);
const name = customer ? customer.name : "occupant";
const plan = customer ? customer.plan : "basic";
const history = customer ? customer.paymentHistory() : [];
```

### Do: a Special Case object with safe behavior

```typescript
class UnknownCustomer implements Customer {
  readonly name = "occupant";
  readonly plan = "basic";
  paymentHistory(): Payment[] {
    return [];
  }
}

const customer = (await customers.find(order.customerId)) ?? new UnknownCustomer();
```


## Plugin

**Problem:** vary implementations without compile-time coupling to the core. **Use when:** deployment-time extension or third-party implementations are real requirements. **Tradeoff:** discovery, compatibility, trust, and debugging become runtime concerns. **Review:** verify a versioned contract, validation, isolation, and actionable load failures; avoid for a fixed implementation set.


## Service Stub

**Problem:** run integration paths without an available or controllable external service. **Use when:** tests or local operation need deterministic boundary behavior. **Tradeoff:** a stub can drift from the real protocol and create false confidence. **Review:** keep it at the actual Gateway boundary and contract-test representative success and failure behavior.


## Record Set

**Problem:** carry and manipulate tabular rows and columns. **Use when:** set-oriented data access or reporting dominates. **Tradeoff:** schema-shaped data leaks broadly and cannot naturally enforce rich behavior. **Review:** validate column and null semantics and do not mistake it for a Domain Model; map to domain objects when invariants matter.
