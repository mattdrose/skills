# Session State

Store only what is needed between requests, with explicit ownership and lifetime.

## Placement criteria

Choose placement by trust, size, durability, latency, revocation, and scaling needs. Prefer stable identifiers over stale object graphs and reload authoritative entities. Session state is never an authorization source by itself.

## Client Session State

**Problem:** Retain state without server storage. **Use when:** it is small, bounded, and safe to return on every request. **Trade-off:** clients can expose, alter, or replay it and payloads grow. **Review:** client-held state must never contain reusable secrets. Validate every return, integrity-sign sensitive data, expire it, and recompute prices and authority server-side. Encryption may protect confidential non-secret values when client placement is unavoidable, but it does not prevent holder theft or replay. Compare server storage for large state or revocation.

### Don't: trust client-held session state

```typescript
app.post("/checkout", (req, res) => {
  const cart = JSON.parse(req.cookies.cart);
  // the client controls the price it pays
  charge(req.user, cart.items, cart.totalCents);
});
```

### Do: sign it and never accept client-computed values

```typescript
app.post("/checkout", async (req, res) => {
  const cart = verifySignedCookie<CartState>(req.cookies.cart, secret); // throws on tamper
  if (cart.expiresAtMs <= Date.now()) throw new ExpiredCartError();
  const items = await catalog.loadCurrentItems(cart.itemIds);
  const totalCents = priceItems(items); // current server data is authoritative
  await charge(req.user, items, totalCents);
  res.sendStatus(204);
});
```

## Server Session State

**Problem:** Keep mutable state off the client with fast access. **Use when:** revocation or frequent updates matter and memory/cache is acceptable. **Trade-off:** affinity, failover, eviction, and memory pressure complicate multiple processes and regions. **Review:** define expiration, revocation, serialization, cache/node-loss behavior, and concurrency. Rotate the session ID after login to prevent fixation; do not park mutable domain graphs in sessions.

### Don't: a server session that never rotates or expires

```typescript
app.post("/login", async (req, res) => {
  const user = await authenticate(req.body);
  // pre-login session id survives, and the entry never expires
  sessions.set(req.cookies.sid ?? newId(), { userId: user.id });
  res.sendStatus(204);
});
```

### Do: rotate the id at login and expire the entry

```typescript
app.post("/login", async (req, res) => {
  const user = await authenticate(req.body);
  const sid = newId(); // fresh id defeats session fixation
  await sessions.set(sid, { userId: user.id }, { ttlSeconds: 3600 });
  res.cookie("sid", sid, { httpOnly: true, secure: true, sameSite: "strict", path: "/" });
  res.sendStatus(204);
});
```

## Database Session State

**Problem:** Share durable state across application nodes. **Use when:** failover and centralized persistence outweigh access cost. **Trade-off:** each access adds latency, database load, contention, and cleanup. **Review:** inspect indexes, TTL cleanup, serialization compatibility, privacy, and hot rows; compare a server cache when durability is unnecessary.

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

## Security and lifecycle review

Treat returned values as untrusted. Re-check current authority server-side; define signing/integrity, confidentiality, expiration, revocation, rotation, and logout behavior. Cookie session identifiers must use `HttpOnly`, `Secure`, and an appropriate `SameSite` policy. Specify concurrent-update semantics and behavior after deployment, cache loss, failover, or schema change. Minimize personal and secret data and bound size.
