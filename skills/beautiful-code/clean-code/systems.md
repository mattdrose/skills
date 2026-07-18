# System Design

System clarity comes from visible policy and constrained dependencies.

- Separate startup and wiring from runtime behavior.
- Keep business policy independent of frameworks, delivery mechanisms, and storage details where
  change warrants it.
- Direct dependencies toward stable concepts; prevent cycles between components.
- Put configuration near the composition root and pass resolved values downward.
- Make cross-cutting behavior—authorization, transactions, observability—consistent and
  discoverable.
- Add extension points in response to demonstrated variation, not imagined futures.
- Evolve design in small, behavior-preserving steps backed by tests.
- Prefer the simplest architecture that meets current reliability and change requirements.

Review architecture through concrete change scenarios: what must know about the change, what can
break, and how will failure be detected?

## Examples

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
