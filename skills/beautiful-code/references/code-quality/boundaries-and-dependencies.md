# Boundaries and Dependencies

Treat user input, files, services, libraries, generated clients, configuration, and persistence records as trust and change boundaries.

## Trust boundaries

- Validate and normalize external data before domain logic sees it.
- Bound payloads, queues, fan-out, and resource use before processing untrusted work.
- Log enough context to diagnose failures, but never log credentials, tokens, or sensitive payloads.

## External systems and vendor types

- Translate foreign types, statuses, errors, and vocabulary into stable application concepts.
- Keep transport, vendor, and storage details out of domain interfaces.
- Concentrate calls to a volatile dependency in one integration layer.
- Wrap a dependency only to constrain its use, translate concepts, enforce policy, or isolate likely change. Do not mirror its API for ceremony.
- Test important assumptions against a real integration or representative contract. Treat malformed responses and upgrades as expected scenarios.
- Set explicit timeouts, cancellation, retry limits, idempotency, and partial-failure behavior.
- Return failures in the caller's vocabulary while preserving the original cause.

## Explicit dependencies

- Expose the capability callers need, not collaborator internals or call chains.
- Preserve one authoritative location for each rule. Deduplicate knowledge, not merely similar syntax.
- Give components focused responsibilities and explicit dependencies. Avoid globals and hidden service lookup.
- Prefer the language or platform when it already solves the problem safely.
- Evaluate maintenance, security history, license, interoperability, operational cost, and exit cost before adding a dependency.
- Use current evidence. Do not build registries, extension points, generic frameworks, or configuration for imagined consumers.

## Reversibility and configuration

- Isolate decisions that are both uncertain and expensive to reverse; keep cheap, stable choices direct.
- Externalize values that legitimately vary by deployment. Validate them at startup, protect secrets, and expose safe effective configuration for diagnosis.
- Keep independent reactions independent. Use events only when their delivery, ordering, duplication, retry, and observability semantics are explicit.

## Examples

### Don't: trust and leak a vendor response

```typescript
async function charge(order: Order): Promise<Stripe.PaymentIntent> {
  return stripe.paymentIntents.create({
    amount: order.totalCents,
    currency: "usd",
    metadata: { orderId: order.id },
  });
}
```

### Do: validate and translate at one adapter

```typescript
type PaymentResult =
  { status: "paid"; providerRef: string } | { status: "declined"; reason: string };

interface PaymentGateway {
  charge(order: Order): Promise<PaymentResult>;
}

class StripePaymentGateway implements PaymentGateway {
  async charge(order: Order): Promise<PaymentResult> {
    const intent = paymentIntentSchema.parse(
      await stripe.paymentIntents.create(
        {
          amount: order.totalCents,
          currency: "usd",
          metadata: { orderId: order.id },
        },
        { timeout: 5_000 },
      ),
    );

    return intent.status === "succeeded"
      ? { status: "paid", providerRef: intent.id }
      : { status: "declined", reason: intent.failureMessage ?? "unknown" };
  }
}
```
