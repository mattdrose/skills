# Distribution

## Placement and boundaries

Distribute only for operational needs such as ownership, scaling, isolation, or geography—not code organization. Every remote boundary adds latency and partial failure; do not disguise a remote call as a local method. Keep boundaries coarse, payloads bounded, and calls observable.

## Remote Facade

**Problem:** Prevent chatty, tightly coupled access to fine-grained remote behavior. **Use when:** clients cross a process or network boundary. **Trade-off:** coarse operations can overfetch, duplicate application APIs, and require versioning. **Review:** count round trips and define timeout, idempotency, authorization, and partial-failure semantics. Replace per-item calls with one operation aligned to the client task.

### Don't: a remote service used like a local object

```typescript
const cart = await remoteCart.getCart(userId);
for (const item of cart.items) {
  // one network call per item, no timeout, retries can double-apply
  const price = await remotePricing.getPrice(item.sku);
  await remoteCart.updateItemPrice(userId, item.id, price);
}
```

### Do: one coarse, failure-aware operation

```typescript
const result = await pricingApi.repriceCart(
  { userId, skus: cart.items.map((i) => i.sku) },
  { timeoutMs: 2000, idempotencyKey: `reprice-${userId}-${cart.version}` },
);

if (result.kind === "timeout") return staleCartWithWarning(cart);
applyRepricedCart(result.cart);
```

## Data Transfer Object

**Problem:** Carry serializable data without exposing internal objects. **Use when:** a remote contract needs explicit shape and batching. **Trade-off:** mapping and evolution add work; oversized DTOs couple unrelated clients. **Review:** require bounded, validated, versionable, behavior-free contracts. Pair them with Remote Facade operations rather than serializing domain graphs, lazy references, or internal fields.

### Don't: serialize domain objects across the boundary

```typescript
app.get("/orders/:id", async (req, res) => {
  const order = await orders.load(req.params.id);
  // internal fields, lazy references, and every refactor leak into the contract
  res.json(order);
});
```

### Do: a bounded, explicit DTO

```typescript
const orderSummarySchema = z.object({
  id: z.string().min(1).max(100),
  customerName: z.string().min(1).max(200),
  lines: z
    .array(
      z.object({
        sku: z.string().min(1).max(100),
        quantity: z.number().int().positive(),
        amountCents: z.number().int().nonnegative(),
      }),
    )
    .max(500),
  status: z.enum(["pending", "shipped", "delivered"]),
});

type OrderSummaryDto = z.infer<typeof orderSummarySchema>;

app.get("/orders/:id", async (req, res) => {
  const order = await orders.load(req.params.id);
  const dto: OrderSummaryDto = orderSummarySchema.parse({
    id: order.id,
    customerName: order.customer.name,
    lines: order.lines.map((line) => ({
      sku: line.sku,
      quantity: line.quantity,
      amountCents: line.amountCents,
    })),
    status: order.status,
  });
  res.json(dto);
});
```

## Timeouts, cancellation, and retries

Every call needs a deadline and useful cancellation propagation. Retry only transient failures, with bounded attempts, backoff, jitter, and an idempotency strategy; a timeout does not prove the remote side did nothing. Do not retry non-idempotent effects blindly. Propagate tracing context and return a deliberate fallback or explicit failure.

## Compatibility and partial failure

Version contracts for rolling deployments and tolerate only intentional forward/backward variation. Validate both incoming and outgoing DTOs and bound payloads. Define what happens when one dependency succeeds and another fails: atomic transaction where possible, otherwise idempotent compensation, durable progress, or a visible partial result. Avoid distributed object graphs and unbounded fan-out.

## Review

Count network calls and payload size; inspect deadlines, cancellation, retry budgets, duplicate-effect protection, contract evolution, and observability. Authenticate and authorize each remote operation at the receiving boundary—network location is not authority. Test timeout, malformed payload, unavailable dependency, stale client, duplicate delivery, and success followed by response loss.
