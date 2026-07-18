# Session State Patterns

Choose placement by trust, size, durability, and scaling needs; session state is not an authorization source by itself.


## Client Session State

**Problem:** retain user state without server-side storage. **Use when:** state is small, bounded, and safe to return on every request. **Tradeoff:** clients can replay or expose it, and payload size grows. **Review:** validate every return, sign integrity-sensitive data, encrypt secrets, expire it, and compare server storage for revocation or large state.

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
app.post("/checkout", (req, res) => {
  const cart = verifySignedCookie<CartState>(req.cookies.cart, secret); // throws on tamper
  const totalCents = priceItems(cart.items);
  charge(req.user, cart.items, totalCents);
});
```


## Server Session State

**Problem:** keep mutable session data off the client with fast access. **Use when:** revocation or frequent updates matter and a cache/memory tier is acceptable. **Tradeoff:** affinity, failover, eviction, and memory pressure complicate scaling. **Review:** verify expiration, revocation, fixation protection, and behavior after node/cache loss; compare database storage for durability.

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
  res.cookie("sid", sid, { httpOnly: true, secure: true });
  res.sendStatus(204);
});
```


## Database Session State

**Problem:** share durable session data across application nodes. **Use when:** failover and centralized persistence outweigh access cost. **Tradeoff:** every session access adds database load, contention, and cleanup work. **Review:** inspect indexes, TTL cleanup, serialization, and hot rows; compare server cache when durability is unnecessary.
