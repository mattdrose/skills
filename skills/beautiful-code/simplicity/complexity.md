# Controlling Complexity

Complexity compounds when features, states, dependencies, and teams must coordinate. Prevent it where possible and reduce it in small, safe steps.

## Common sources

- Expanding a system beyond its core purpose.
- Adding features or configuration without demonstrated value.
- Coupling unrelated responsibilities or volatile technologies.
- Choosing dependencies with weak maintenance, quality, or interoperability.
- Reimplementing capabilities that a suitable standard or maintained dependency provides.
- Encoding the same knowledge in multiple places.
- Building without understanding the problem or existing system.

When a solution becomes difficult, step back and restate the underlying problem without reference to the current implementation. Complexity at one level often reveals a poor decision beneath it.

### Don't: reimplement a capability the platform provides

```typescript
// Hand-rolled parsing that URL and URLSearchParams already do, with edge cases this gets wrong.
function parseQuery(url: string): Record<string, string> {
  const query = url.split("?")[1] ?? "";
  const result: Record<string, string> = {};
  for (const pair of query.split("&")) {
    const [key, value] = pair.split("=");
    if (key) result[key] = decodeURIComponent(value ?? "");
  }
  return result;
}
```

### Do: use the platform

```typescript
function parseQuery(url: string): Record<string, string> {
  return Object.fromEntries(new URL(url).searchParams);
}
```

### Don't: add configuration without demonstrated value

```typescript
// Nobody asked for these knobs; every option multiplies the states to test and support.
interface RetryOptions {
  attempts?: number;
  backoff?: "fixed" | "linear" | "exponential";
  jitter?: boolean;
  retryIf?: (error: Error) => boolean;
  onRetry?: (attempt: number, error: Error) => void;
}

async function fetchWithRetry(url: string, opts: RetryOptions = {}): Promise<Response> {
  // ...must handle every combination above
}
```

### Do: support only the behavior the system needs

```typescript
async function fetchWithRetry({
  url,
  attempts = 3,
}: {
  url: string;
  attempts?: number;
}): Promise<Response> {
  let lastError: unknown;
  for (let i = 0; i < attempts; i++) {
    try {
      return await fetch(url);
    } catch (error) {
      lastError = error;
    }
  }
  throw lastError;
}
```

## Reduction strategy

1. Make behavior observable with tests or other verification.
2. Improve readability enough to see responsibilities and coupling.
3. Simplify one piece without changing behavior.
4. Deliver the needed feature or fix.
5. Repeat where evidence justifies it.

Hide unavoidable external complexity behind the narrowest useful boundary. Do not hide complexity created by the application itself; remove it.

### Don't: let a vendor API leak into every caller

```typescript
// Every caller now depends on Stripe's types, statuses, and error shapes.
async function checkout(cart: Cart, stripe: Stripe): Promise<Stripe.PaymentIntent> {
  return stripe.paymentIntents.create({
    amount: cart.totalCents,
    currency: "usd",
    payment_method: cart.paymentMethodId,
    confirm: true,
  });
}
```

### Do: hide the external complexity behind the narrowest useful boundary

```typescript
type PaymentResult = { status: "paid"; receiptId: string } | { status: "declined"; reason: string };

async function takePayment(cart: Cart): Promise<PaymentResult> {
  const intent = await stripe.paymentIntents.create({
    amount: cart.totalCents,
    currency: "usd",
    payment_method: cart.paymentMethodId,
    confirm: true,
  });
  return intent.status === "succeeded"
    ? { status: "paid", receiptId: intent.id }
    : {
        status: "declined",
        reason: intent.last_payment_error?.message ?? "unknown",
      };
}
```
