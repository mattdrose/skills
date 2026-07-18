# Distribution Patterns

Distribution adds latency and partial failure; use boundaries coarse enough to expose those facts.


## Remote Facade

**Problem:** prevent chatty, tightly coupled remote access to fine-grained behavior. **Use when:** clients cross a process or network boundary. **Tradeoff:** coarse operations can overfetch, duplicate application APIs, and require versioning. **Review:** count round trips and verify timeout, idempotency, authorization, and partial-failure semantics.

### Don't: fine-grained remote calls

```typescript
// four round trips to render one screen
const order = await api.getOrder(orderId);
const customer = await api.getCustomer(order.customerId);
const lines = await api.getOrderLines(orderId);
const shipping = await api.getShippingStatus(orderId);
render(customer.name, lines, shipping);
```

### Do: one coarse facade operation

```typescript
// one round trip; the facade assembles the data server-side
const summary = await api.getOrderSummary(orderId);

render({
  name: summary.customerName,
  lines: summary.lines,
  status: summary.shippingStatus,
});
```


## Data Transfer Object

**Problem:** carry serializable data across a boundary without exposing internal objects. **Use when:** a remote contract needs explicit shape and batching. **Tradeoff:** mapping and contract evolution add work, while oversized DTOs couple unrelated clients. **Review:** require bounded, validated, versionable, behavior-free contracts; pair with Remote Facade operations rather than mirroring the domain graph.

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
interface OrderSummaryDto {
  id: string;
  customerName: string;
  lines: { sku: string; quantity: number; amountCents: number }[];
  status: "pending" | "shipped" | "delivered";
}

app.get("/orders/:id", async (req, res) => {
  const order = await orders.load(req.params.id);
  res.json(validateOrder(order));
});
```
