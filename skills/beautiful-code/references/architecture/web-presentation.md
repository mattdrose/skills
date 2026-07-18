# Web Presentation

Choose controller and view patterns independently; keep request handling, navigation, rendering, persistence, and domain policy distinct.

## Model View Controller

**Problem:** Separate state/policy, input handling, and rendering. **Use when:** interactive presentation benefits from independently changing roles. **Trade-off:** framework terminology can hide responsibilities and add indirection to simple pages. **Review:** trace an interaction: the model owns state and policy, controller interprets input, and view renders.

## Page Controller

**Problem:** Handle each page or endpoint directly. **Use when:** request flows are independent and simple. **Trade-off:** shared authentication, parsing, and response behavior can duplicate and drift. **Review:** keep endpoint behavior local; use Front Controller only for truly shared concerns.

## Front Controller

**Problem:** Apply routing and cross-cutting request behavior consistently. **Use when:** requests share authentication, parsing, or dispatch. **Trade-off:** it can become a bottleneck or god object. **Review:** limit it to shared preprocessing and dispatch; do not absorb page or domain behavior. Centralizing authentication prevents endpoint omissions.

### Don't: authentication duplicated in every page controller

```typescript
app.get("/orders", async (req, res) => {
  const user = await verifyToken(req.headers.authorization);
  if (!user) return res.sendStatus(401);
  res.json(await listOrders(user));
});

app.get("/invoices", async (req, res) => {
  const user = await verifyToken(req.headers.authorization); // copied per endpoint, drifts
  if (!user) return res.sendStatus(401);
  res.json(await listInvoices(user));
});
```

### Do: shared preprocessing at the front controller, page logic local

```typescript
app.use(async (req, res, next) => {
  req.user = await verifyToken(req.headers.authorization);
  if (!req.user) return res.sendStatus(401);
  next();
});

app.get("/orders", async (req, res) => res.json(await listOrders(req.user)));
app.get("/invoices", async (req, res) => res.json(await listInvoices(req.user)));
```

## Template View

**Problem:** Render output conveniently from dynamic data. **Use when:** contributors work naturally in the output format. **Trade-off:** templates invite domain decisions and context-sensitive injection. **Review:** require contextual output encoding and presentation-only conditionals over prepared data; never render raw request input.

### Don't: business decisions inside the template

```typescript
const html = `
  <p>Total: ${formatCents(order.totalCents)}</p>
  ${
    order.totalCents > 50_000 && customer.tier === "gold"
      ? `<p>Discount: ${formatCents(Math.round((order.totalCents * 500) / 10_000))}</p>` // pricing policy lives in markup
      : ""
  }
`;
```

### Do: presentation-only conditionals over prepared data

```typescript
const view = presentOrder(order, customer); // the model already computed any discount

const html = `
  <p>Total: ${escapeHtml(view.totalFormatted)}</p>
  ${view.discountFormatted ? `<p>Discount: ${escapeHtml(view.discountFormatted)}</p>` : ""}
`;
```

## Transform View

**Problem:** Convert data to output in one transformation. **Use when:** output is structured and deterministic. **Trade-off:** transformations can be harder to author incrementally and can mix shaping with rendering. **Review:** pass presentation-ready input and encode output correctly; prefer Template View when template-centric editing matters.

## Two Step View

**Problem:** Apply consistent framing across many views. **Use when:** pages share layout but render distinct content. **Trade-off:** two stages complicate data flow, escaping, and debugging. **Review:** make stage one emit a clear logical page and stage two apply only shared layout; define where encoding occurs to avoid missed or double escaping.

## Application Controller

**Problem:** Coordinate navigation across related requests. **Use when:** multi-step flows share state and transition rules. **Trade-off:** centralization can absorb domain policy or unrelated flows. **Review:** keep explicit flow transitions here and business invariants in the domain; scattered handlers otherwise re-derive and drift in navigation.

### Don't: navigation rules scattered across handlers

```typescript
app.post("/signup/plan", async (req, res) => {
  await savePlan(req);
  // each handler re-derives the flow
  if (req.body.plan === "team") res.redirect("/signup/seats");
  else if (req.session.needsBilling) res.redirect("/signup/billing");
  else res.redirect("/signup/confirm");
});
```

### Do: an application controller owning the flow

```typescript
const signupFlow = new FlowController<SignupState>({
  plan: (s) => (s.plan === "team" ? "seats" : s.needsBilling ? "billing" : "confirm"),
  seats: (s) => (s.needsBilling ? "billing" : "confirm"),
  billing: () => "confirm",
});

app.post("/signup/:step", async (req, res) => {
  const state = await saveStep(req);
  res.redirect(`/signup/${signupFlow.next(req.params.step, state)}`);
});
```

## Security review

Validate all untrusted input, enforce authorization at each application-operation boundary, and contextually encode output. With cookie authentication, use safe HTTP methods for reads; require CSRF tokens or equivalent origin-bound defenses for state changes; and set an appropriate `SameSite` cookie policy. Check idempotency where clients may repeat requests. Framework pattern names do not guarantee separation or security; reject persistence or domain policy in controllers and views.
