# Design and Delivery

## Easy-to-change design

**Review question:** Does the design make likely changes local and unsurprising?

Prefer structures that reduce coupling and cognitive load. A good design is easier to adapt than its alternatives; elegance without changeability is not enough.

## Duplication

**Review question:** Is one piece of knowledge represented in more than one place?

Keep each rule, fact, and decision authoritative in one place. Not all similar-looking code shares knowledge, and forced reuse can couple unrelated concepts. Remove duplication of intent, not merely repeated syntax.

### Don't: encode one business rule in two places

```typescript
function displayTotal(order: Order): string {
  return `$${(order.subtotal * 1.13).toFixed(2)}`; // tax rate also hardcoded in chargeCustomer
}

function chargeCustomer(order: Order): number {
  return Math.round(order.subtotal * 1.13 * 100);
}
```

### Do: keep the rule authoritative in one place

```typescript
const HST_RATE = 0.13;

function totalWithTax(order: Order): number {
  return order.subtotal * (1 + HST_RATE);
}

function displayTotal(order: Order): string {
  return `$${totalWithTax(order).toFixed(2)}`;
}

function chargeCustomer(order: Order): number {
  return Math.round(totalWithTax(order) * 100);
}
```

## Orthogonality

**Review question:** Can one concern change without unrelated parts moving with it?

Give components focused responsibilities and narrow interfaces. Avoid global state, hidden dependencies, and chains that expose internals. Independent parts are easier to test, replace, and reason about.

### Don't: let one concern depend on another's internals

```typescript
function sendReceipt(orderId: string) {
  const row = db.orders.rawRow(orderId); // email formatting now moves whenever the schema does
  mailer.send(row.cust_email_addr, `Total: $${row.amt_cents / 100}`);
}
```

### Do: communicate through a narrow interface

```typescript
interface Receipt {
  email: string;
  totalFormatted: string;
}

function sendReceipt(receipt: Receipt) {
  mailer.send({ to: receipt.email, message: `Total: ${receipt.totalFormatted}` });
}
```

## Reversibility

**Review question:** Which decisions are expensive to undo, and can they be deferred or isolated?

Assume requirements, vendors, and architectures will change. Put volatile choices behind clear boundaries and avoid encoding one deployment or workflow everywhere. Preserve options where uncertainty is material.

### Don't: spread a volatile vendor choice through every call site

```typescript
// every module builds Twilio payloads directly; switching providers touches all of them
await twilio.messages.create({ from: TWILIO_FROM, to: user.phone, body: renderAlert(alert) });
await twilio.messages.create({ from: TWILIO_FROM, to: user.phone, body: renderReceipt(order) });
```

### Do: isolate the reversible decision behind a boundary

```typescript
interface Notifier {
  send({ to, message }: { to: User; message: string }): Promise<void>;
}

const notifier: Notifier = new TwilioNotifier(twilio, TWILIO_FROM);
await notifier.send({ to: user, message: renderAlert(alert) });
await notifier.send({ to: user, message: renderReceipt(order) });
```

## Tracer bullets

**Review question:** Has a thin, production-shaped path validated the architecture end to end?

Implement a narrow slice through real layers early. It should compile, integrate, and provide feedback while leaving room to grow. Unlike a disposable prototype, the path becomes part of the product.

### Don't: build complete layers before anything runs end to end

```typescript
class ReportExporter {
  // handles every format and layout before a single report has ever been served
  async export(id: string, format: "pdf" | "csv" | "xlsx", layout: Layout): Promise<Buffer> {
    const report = await this.repo.findById(id);
    return this.renderers[format].render(report, layout);
  }
}
```

### Do: ship a thin production-shaped slice first

```typescript
// minimal but real: route, storage, and rendering integrated and demonstrable now
app.get("/reports/:id.pdf", async (req, res) => {
  const report = await reportStore.findById(req.params.id);
  res.type("application/pdf").send(renderPdf(report));
});
```

## Prototypes

**Review question:** Is uncertainty being tested cheaply before production commitments are made?

Prototype risky algorithms, interfaces, or workflows while deliberately ignoring irrelevant details. State what the experiment must answer and whether its code will be discarded. Never let exploratory shortcuts enter production by accident.

## Domain languages

**Review question:** Does the representation express domain intent without unnecessary implementation detail?

Use the vocabulary and notation of the problem. A small API, data format, or existing language is often enough; build a custom parser only when its value justifies the maintenance and tooling cost.

## Estimation

**Review question:** Does the estimate expose assumptions, uncertainty, and the decision it supports?

Choose precision appropriate to the horizon. Model the system, derive a range from known quantities and comparable work, and refine it with measured progress. Treat estimates as feedback, not promises disguised as numbers.
