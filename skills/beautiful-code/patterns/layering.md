# Layering

Separate presentation, domain, and data-source concerns so each can evolve independently.

## Prefer

Dependencies point inward toward domain policy; presentation translates interaction, and data-source code translates persistence.

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

## Review

Look for business rules in controllers/templates, SQL in domain objects without an Active Record choice, and layers that only forward calls.

## Trade-offs

Boundaries improve replaceability and testing, but extra translation and pass-through layers add cost. Add a boundary for a real axis of change, not symmetry.

## Layer vs. tier

A layer is a logical boundary. A tier is a deployment boundary. Do not introduce network calls merely to preserve layering.
