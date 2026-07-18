# Adaptable Architecture

## Decoupling

**Review question:** Does a component know more about its collaborators than its task requires?

Minimize dependency reach and expose behavior through narrow interfaces. Avoid train-wreck call chains, global access, and APIs that leak internal structure. Couple to stable capabilities rather than concrete organization.

### Don't: chain through collaborators' internals

```typescript
function applyDiscount(customer: Customer, amount: number) {
  // caller now depends on the shape of account, orders, and totals
  customer.getAccount().getOrders().last().getTotals().setDiscount(amount);
}
```

### Do: ask for the behavior, not the structure

```typescript
function applyDiscount({ customer, amount }: { customer: Customer; amount: number }) {
  customer.applyDiscountToLatestOrder(amount);
}
```

## Events

**Review question:** Would a state change be clearer if producers did not control every reaction?

Use events when multiple independent consumers need notification. Choose direct callbacks, streams, queues, or event logs according to delivery and ordering needs. Make retries, duplication, failure, and observability explicit.

### Don't: make the producer orchestrate every reaction

```typescript
async function completeOrder(order: Order) {
  await db.orders.markComplete(order.id);
  await emailService.sendReceipt(order); // every new reaction means editing this function
  await analytics.track("order_completed", order);
  await warehouse.scheduleShipment(order);
}
```

### Do: publish the state change and let consumers subscribe

```typescript
async function completeOrder(order: Order) {
  await db.orders.markComplete(order.id);
  await events.publish("order.completed", { orderId: order.id });
}
// receipts, analytics, and shipping each subscribe independently
```

## Transformation pipelines

**Review question:** Can the work be understood as data moving through small transformations?

Express processing as a sequence from input to output when that model fits. Keep intermediate representations explicit and transformations composable. Avoid hiding essential state changes inside an apparently functional pipeline.

## Composition over inheritance

**Review question:** Is inheritance being used for substitutability, or merely to reuse implementation?

Prefer interfaces, delegation, components, and mixins that assemble behavior without binding classes into a rigid hierarchy. Use inheritance only when the subtype relationship is stable and callers can safely treat every subtype as its base.

### Don't: reuse implementation through inheritance

```typescript
class CsvImporter extends HttpClient {
  // inherits retry(), auth(), and a dozen methods it must never expose
  async import(url: string): Promise<Row[]> {
    return parseCsv(await this.get(url));
  }
}
```

### Do: compose the capability you need

```typescript
class CsvImporter {
  constructor(private readonly fetchText: (url: string) => Promise<string>) {}

  async import(url: string): Promise<Row[]> {
    return parseCsv(await this.fetchText(url));
  }
}
```

## Configuration

**Review question:** Can operational policy change without rebuilding the application?

Externalize values that legitimately vary by environment or deployment. Validate configuration at startup, secure secrets, and make effective values observable. Do not turn every constant into a setting; excess configuration creates another language to maintain.

### Don't: hardcode deployment policy into the build

```typescript
function paymentsClient() {
  // pointing staging at its own gateway requires editing code and rebuilding
  return new PaymentsClient("https://payments.prod.example.com", "sk_live_9f3a");
}
```

### Do: externalize and validate configuration at startup

```typescript
const config = loadConfig(); // fails at startup if PAYMENTS_URL or PAYMENTS_KEY is missing

function paymentsClient() {
  return new PaymentsClient(config.paymentsUrl, config.paymentsKey);
}
```
