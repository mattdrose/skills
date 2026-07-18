# Session State

Store only the state needed between requests, with explicit ownership and lifetime.

## Client state

Scales naturally but must be size-limited, integrity-protected, and free of secrets. Treat every returned value as untrusted.

### Don't: authorize from a client-held session value

```typescript
app.post("/admin/refunds", (req, res) => {
  const session = JSON.parse(req.cookies.session);
  if (session.role !== "admin") return res.sendStatus(403); // the client wrote this value
  issueRefund(req.body.orderId);
});
```

### Do: verify integrity and re-check authority server-side

```typescript
app.post("/admin/refunds", async (req, res) => {
  const session = verifySignedCookie<Session>(req.cookies.session, secret); // throws on tamper
  const user = await users.byId(session.userId);

  if (!user.hasRole("admin")) return res.sendStatus(403);

  issueRefund(req.body.orderId);
});
```

## Server state

Simple and private, but needs expiration, fixation protection, and a strategy for multiple processes or regions.

### Don't: an object graph parked in the session

```typescript
await session.set("checkout", {
  order, // entire entity, lines and customer included; stale after any other write
  customer: order.customer,
  appliedPromos: promoEngine.evaluate(order),
});
```

### Do: stable identifiers, reload on each request

```typescript
await session.set("checkout", {
  orderId: order.id,
  promoCode: promo?.code ?? null,
});

// next request
const { orderId, promoCode } = await session.get("checkout");
const order = await orders.byId(orderId); // always current, survives deploys
```

## Database state

Durable and shareable, at the cost of latency, cleanup work, and contention.

## Review

Prefer stable identifiers over object graphs. Define expiration, revocation, concurrency behavior, privacy, and behavior after deployment or failover.
