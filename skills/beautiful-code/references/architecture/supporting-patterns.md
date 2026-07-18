# Supporting Patterns

Retain a supporting pattern only while its boundary or semantics remain visible.

## Boundaries and mapping

### Gateway

**Problem:** Isolate an external protocol or vendor API. **Use when:** application code needs a stable application-oriented boundary. **Trade-off:** a leaky or generic wrapper adds indirection without insulation. **Review:** stop protocol details, vendor states, failure translation, timeouts, and retries at the Gateway. Treat it as a trust boundary: validate untrusted input/output, authenticate and authorize operations, constrain payloads, and avoid leaking secrets.

### Mapper

**Problem:** Translate independent representations. **Use when:** neither side should depend on the other. **Trade-off:** duplicated fields and lossy transformations can drift silently. **Review:** inspect round-trip semantics, defaults, validation, and errors; omit it when representations are intentionally identical and coupled.

### Layer Supertype

**Problem:** Share behavior required by every type in a layer. **Use when:** a genuine layer-wide contract or implementation exists. **Trade-off:** empty bases and broad inheritance create coupling. **Review:** require meaningful shared behavior; delete naming-only supertypes.

### Separated Interface

**Problem:** Reverse a package dependency while hiding implementation. **Use when:** policy should own an abstraction implemented by infrastructure. **Trade-off:** extra interfaces fragment navigation or duplicate one class. **Review:** verify dependency direction actually changes and consumer vocabulary shapes the interface.

### Registry

**Problem:** Make shared process objects globally reachable. **Use when:** explicit passing is genuinely impractical for stable infrastructure. **Trade-off:** hidden dependencies, mutable global state, lifetime ambiguity, and test interference. **Review:** scope lifetime and make registration immutable. Do not use Registry as a global service locator for ordinary dependencies; prefer explicit injection.

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

## Domain values and special behavior

### Value Object

**Problem:** Model a concept defined by attributes rather than identity. **Use when:** equality and operations depend only on value. **Trade-off:** mutability breaks equality and persistence assumptions. **Review:** require validated construction, value equality, and practical immutability; use an entity when lifecycle identity matters.

### Money

**Problem:** Preserve currency and arithmetic rules. **Use when:** amounts are compared, rounded, allocated, or converted. **Trade-off:** naive numbers and implicit conversion cause financial errors. **Review:** require currency-aware equality, decimal precision, explicit rounding and exchange rates; never use a bare float.

#### Don't: monetary math on bare floats

```typescript
function applyDiscount(invoice: { total: number }, percent: number): number {
  return invoice.total * (1 - percent / 100); // float drift, no currency, silent rounding
}

const owed = applyDiscount({ total: 19.99 }, 15) + 4.95;
```

#### Do: a Money value object

```typescript
class Money {
  readonly cents: bigint;
  readonly currency: string;

  constructor({ cents, currency }: { cents: bigint; currency: string }) {
    const normalizedCurrency = currency.toUpperCase();
    if (!/^[A-Z]{3}$/.test(normalizedCurrency)) throw new Error("invalid currency");

    this.cents = cents;
    this.currency = normalizedCurrency;
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

### Special Case

**Problem:** Remove repeated handling of a common exceptional value. **Use when:** absence has stable, safe behavior. **Trade-off:** it can hide missing-data errors or invent misleading defaults. **Review:** ensure behavior is semantically valid and remains distinguishable where required; use explicit errors for unexpected absence.

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

## Extension and construction

### Plugin

**Problem:** Vary implementations without compile-time coupling to core. **Use when:** deployment-time or third-party extension is real. **Trade-off:** discovery, compatibility, trust, and debugging move to runtime. **Review:** require a versioned contract, validation, isolation, least privilege, and actionable load failures; avoid it for a fixed set.

## Testing and data access

### Service Stub

**Problem:** Run integration paths without an available or controllable external service. **Use when:** tests or local operation need deterministic boundary behavior. **Trade-off:** stubs drift and create false confidence. **Review:** place it at the real Gateway boundary and contract-test representative success, timeout, malformed, and failure behavior.

### Record Set

**Problem:** Carry and manipulate rows and columns. **Use when:** set-oriented access or reporting dominates. **Trade-off:** schema-shaped data leaks and cannot naturally enforce rich behavior. **Review:** validate column and null semantics; do not mistake it for a Domain Model, and map to domain objects where invariants matter.
