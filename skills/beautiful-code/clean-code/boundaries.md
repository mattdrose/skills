# Boundaries

External libraries, services, files, and user input are trust and change boundaries.

- Concentrate vendor-specific types and calls in a narrow integration layer.
- Translate external data into validated internal forms before domain logic uses it.
- Define timeouts, cancellation, retries, idempotency, and partial-failure behavior explicitly.
- Test assumptions about third-party behavior with a real integration or representative contract.
- Avoid leaking transport details, generated clients, or persistence records through the system.
- Wrap an API only when the wrapper narrows usage, translates concepts, or isolates likely change—not merely to rename every method.
- Treat version upgrades and malformed responses as expected maintenance scenarios.
- Log boundary failures with useful context while excluding secrets and sensitive data.

A good boundary limits both blast radius and the amount of foreign knowledge the rest of the code must carry.

## Examples

### Don't: leak vendor types through the domain

```typescript
import Stripe from "stripe";

// Every caller now depends on Stripe's types, statuses, and failure modes
export async function chargeOrder(order: Order, stripe: Stripe): Promise<Stripe.PaymentIntent> {
  return stripe.paymentIntents.create({
    amount: order.totalCents,
    currency: "usd",
    metadata: { orderId: order.id },
  });
}
```

### Do: translate at a narrow adapter

```typescript
// shared vendor-agnostic interfaces
export interface PaymentResult {
  status: "SUCCEEDED" | "REQUIRES_ACTION" | "DECLINED";
  providerRef: string;
}

export interface PaymentGateway {
  charge(order: Order): Promise<PaymentResult>;
}

// One adapter owns the vendor types, timeouts, and status translation
export class StripePaymentGateway implements PaymentGateway {
  constructor(private readonly stripe: Stripe) {}

  async charge(order: Order): Promise<PaymentResult> {
    const intent = await this.stripe.paymentIntents.create(
      {
        amount: order.totalCents,
        currency: "usd",
        metadata: { orderId: order.id },
      },
      { timeout: 5_000 },
    );

    return {
      status: toPaymentStatus(intent.status),
      providerRef: intent.id,
    };
  }
}
```

### Don't: trust an external response to match your types

```typescript
async function fetchRates(): Promise<RateTable> {
  const res = await fetch(RATES_URL);
  // the cast is a wish; malformed data fails far away
  return (await res.json()) as RateTable;
}
```

### Do: validate external data before the domain sees it

```typescript
async function fetchRates(): Promise<RateTable> {
  const res = await fetch(RATES_URL);

  const parsed = rateTableSchema.safeParse(await res.json());
  if (!parsed.success) {
    throw new RateFeedError("Malformed rate feed", { cause: parsed.error });
  }

  return parsed.data;
}
```
