# Layering and Domain Logic

## Dependency direction

Separate presentation, domain, and data-source concerns where they change independently. Dependencies point inward toward stable domain policy: presentation translates interactions and data access translates persistence. Keep startup, framework wiring, environment configuration, and cross-cutting policy such as observability visible, consistent, and discoverable at the composition root; pass resolved dependencies downward and prevent cycles. A layer is logical, while a tier is a deployment boundary—do not add a network hop for layering symmetry.

Boundaries improve testing and replaceability but translation and pass-through layers cost navigation. Add extension points and layers for demonstrated variation, reliability, or change, not imagined futures.

### Don't: let domain logic reach for the environment and build its own wiring

```typescript
class InvoiceService {
  async send(invoice: Invoice): Promise<void> {
    const mailer = new SendgridMailer(process.env.SENDGRID_KEY!);
    const template = process.env.FEATURE_NEW_TEMPLATE === "true" ? "v2" : "v1";
    await mailer.send(renderInvoice(invoice, template));
  }
}
```

### Do: resolve configuration at the composition root and pass values down

```typescript
// composition root
const mailer = new SendgridMailer(config.sendgridKey);
const invoiceService = new InvoiceService({
  mailer,
  template: config.invoiceTemplate,
});

class InvoiceService {
  private readonly mailer: Mailer;
  private readonly template: TemplateName;

  constructor({ mailer, template }: { mailer: Mailer; template: TemplateName }) {
    this.mailer = mailer;
    this.template = template;
  }

  async send(invoice: Invoice): Promise<void> {
    await this.mailer.send(renderInvoice(invoice, this.template));
  }
}
```

### Don't: trap business policy inside the delivery mechanism

```typescript
app.post("/refunds", async (req, res) => {
  const order = await ordersCollection.findOne({
    _id: new ObjectId(req.body.orderId),
  });
  // the refund-window rule is welded to Express and MongoDB
  if (Date.now() - order.paidAtMs > 30 * 24 * 60 * 60 * 1000) {
    return res.status(422).json({ error: "Refund window closed" });
  }
  await issueRefund(order);
  res.status(204).end();
});
```

### Do: keep the policy in plain domain code the route calls

```typescript
const REFUND_WINDOW_MS = 30 * 24 * 60 * 60 * 1000;

export function isRefundable({ order, nowMs }: { order: Order; nowMs: number }): boolean {
  return nowMs - order.paidAtMs <= REFUND_WINDOW_MS;
}

app.post("/refunds", async (req, res) => {
  const order = await orders.byId(req.body.orderId);
  if (!isRefundable({ order, nowMs: Date.now() })) {
    return res.status(422).json({ error: "Refund window closed" });
  }

  await issueRefund(order);
  res.status(204).end();
});
```

### Don't: business rules embedded in the HTTP handler

```typescript
app.post("/refunds", async (req, res) => {
  const order = await db.orders.findById(req.body.orderId);
  if (order.shippedAt && daysSince(order.shippedAt) > 30) {
    return res.status(422).json({ error: "too late" }); // refund policy lives in the handler
  }
  const amountCents = order.totalCents - (order.usedPromo ? order.promoCents : 0);
  await db.refunds.insert({ orderId: order.id, amountCents });
  res.json({ amountCents });
});
```

### Do: the handler translates; the domain decides

```typescript
app.post("/refunds", async (req, res) => {
  const result = await refundService.requestRefund(req.body.orderId);

  if (result.kind === "rejected") {
    return res.status(422).json({ error: result.reason });
  }

  res.json({ amountCents: result.refund.amount.cents });
});

class RefundService {
  async requestRefund(orderId: string): Promise<RefundResult> {
    const order = await this.orders.byId(orderId);
    return order.refund(this.clock.now()); // eligibility and amount rules live here
  }
}
```

### Don't: SQL inside a domain object without choosing Active Record

```typescript
class Order {
  async applyDiscount(percent: number): Promise<void> {
    this.totalCents = Math.round(this.totalCents * (1 - percent / 100));
    // domain rules now require a live database to test
    await db.query("UPDATE orders SET total_cents = $1 WHERE id = $2", [this.totalCents, this.id]);
  }
}
```

### Do: the domain decides; data-source code persists

```typescript
class Order {
  applyDiscount(percent: number): void {
    this.totalCents = Math.round(this.totalCents * (1 - percent / 100));
  }
}

class OrderMapper {
  async save(order: Order): Promise<void> {
    await db.query("UPDATE orders SET total_cents = $1 WHERE id = $2", [
      order.totalCents,
      order.id,
    ]);
  }
}
```

## Transaction Script

**Problem:** Organize one business operation. **Use when:** workflows and rules are few and simple. **Trade-off:** scripts duplicate rules and grow conditional tangles. **Review:** repeated policy and cross-script invariants indicate a Domain Model.

## Table Module

**Problem:** Organize logic around tabular data. **Use when:** record-set tools and a table-shaped domain dominate. **Trade-off:** row-spanning logic is convenient, but rich object behavior and cross-table rules become awkward. **Review:** ensure table boundaries match rules; otherwise compare Transaction Script or Domain Model.

## Domain Model

**Problem:** Represent interacting business rules and invariants. **Use when:** behavior is complex enough to justify behavior-rich, identity-bearing domain objects. **Trade-off:** mapping and orchestration cost more than scripts or modules. **Review:** keep rules in the model and persistence, transport, environment access, and transaction control outside. Duplicated pricing or eligibility logic across routes and scripts will drift.

### Don't: rules duplicated across transaction scripts

```typescript
async function placeOrder(order: OrderRow): Promise<void> {
  if (order.totalCents > 50_000 && order.customerTier === "gold") {
    order.totalCents = Math.round((order.totalCents * 9_500) / 10_000);
  }
  await db.orders.insert(order);
}

async function quoteOrder(order: OrderRow): Promise<number> {
  if (order.totalCents > 50_000 && order.customerTier === "gold") {
    order.totalCents = Math.round((order.totalCents * 9_500) / 10_000); // copy drifts
  }
  return order.totalCents;
}
```

### Do: the rule lives once, in the domain model

```typescript
class Order {
  private lines: OrderLine[];
  private customer: Customer;

  constructor({ lines, customer }: { lines: OrderLine[]; customer: Customer }) {
    this.lines = lines;
    this.customer = customer;
  }

  total(): Money {
    const gross = this.lines.reduce((sum, l) => sum.add(l.amount), Money.zero("USD"));
    return this.customer.qualifiesForVolumeDiscount(gross)
      ? gross.timesRatio({ numerator: 95n, denominator: 100n })
      : gross;
  }
}

const placed = await orders.save(order); // both paths call order.total()
const quoted = order.total();
```

## Service Layer

**Problem:** Expose stable application operations and transaction boundaries. **Use when:** clients need coordinated authorization, transactions, and domain calls. **Trade-off:** it can become an anemic-domain dumping ground or needless forwarding layer. **Review:** expect orchestration and authorization here, core rules in the domain, and remove services that only rename repository calls.

### Don't: a service layer that only forwards calls

```typescript
class OrderService {
  getOrder(id: string) {
    return this.repo.findById(id);
  }
  saveOrder(order: Order) {
    return this.repo.save(order);
  }
  cancelOrder(id: string) {
    return this.repo.cancel(id);
  } // pure indirection, no coordination
}
```

### Do: a service owning transaction and authorization, rules in the domain

```typescript
class OrderService {
  async cancelOrder({ user, id }: { user: User; id: string }): Promise<void> {
    await this.tx.run(async () => {
      const order = await this.repo.findById(id);
      this.authz.require(user, "order:cancel", order);
      order.cancel(); // invariant checks live on Order

      await this.repo.save(order);
    });
  }
}
```

### Don't: every rule piles into the service layer

```typescript
class SubscriptionService {
  async renew(id: string): Promise<void> {
    const sub = await this.repo.byId(id);
    if (sub.status === "canceled") throw new Error("cannot renew");
    if (sub.plan === "annual") sub.expiresAt = addYears(sub.expiresAt, 1);
    else sub.expiresAt = addMonths(sub.expiresAt, 1);
    sub.renewalCount += 1; // domain rules stranded outside the domain type
    await this.repo.save(sub);
  }
}
```

### Do: the service coordinates; the domain type owns the rules

```typescript
class SubscriptionService {
  async renew(id: string): Promise<void> {
    const sub = await this.repo.byId(id);
    sub.renew();
    await this.repo.save(sub);
  }
}

class Subscription {
  renew(): void {
    if (this.status === "canceled") throw new CannotRenewError(this.id);
    this.expiresAt = this.plan.extend(this.expiresAt);
    this.renewalCount += 1;
  }
}
```

## Choosing the simplest model

Start with Transaction Script for simple operations, Table Module for record-oriented logic, and Domain Model for interacting invariants. Introduce Service Layer only where operation coordination is real. Evolve in small behavior-preserving steps backed by tests. Review concrete change scenarios: what must know, what can break, and how failure is detected.

## Review

Find business policy in controllers or templates, SQL in domain objects without an explicit Active Record choice, direct environment/framework dependencies in policy, cycles, inconsistent authorization or transactions, and layers that only forward. Identify invariants and transaction boundaries, then trace a read and write. Prefer the simplest architecture meeting current change and reliability needs.
