# Web Presentation Patterns

Choose controller and view patterns independently; keep domain policy out of both.


## Model View Controller

**Problem:** separate state/policy, input handling, and rendering. **Use when:** an interactive presentation benefits from independently changing roles. **Tradeoff:** framework terminology can obscure responsibilities and add indirection to simple pages. **Review:** trace one interaction and reject domain rules or persistence in controllers and views.


## Page Controller

**Problem:** handle each page or endpoint directly. **Use when:** request flows are mostly independent and simple. **Tradeoff:** authorization, parsing, and response behavior duplicate across controllers. **Review:** compare Front Controller for truly shared request concerns, but keep endpoint-specific logic local.


## Front Controller

**Problem:** apply routing and cross-cutting request behavior consistently. **Use when:** all requests share authentication, parsing, or dispatch. **Tradeoff:** the entry point can become a bottleneck and god object. **Review:** keep it to shared preprocessing and dispatch; leave page behavior to Page Controllers.

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

**Problem:** render output conveniently from dynamic data. **Use when:** designers or developers work naturally in the output format. **Tradeoff:** templates invite business logic and context-sensitive injection bugs. **Review:** verify contextual escaping and presentation-only conditionals; compare Transform View for deterministic programmatic conversion.

### Don't: business decisions inside the template

```typescript
const html = `
  <p>Total: ${formatCents(order.totalCents)}</p>
  ${
    order.totalCents > 500_00 && customer.tier === "gold"
      ? `<p>Discount: ${formatCents(order.totalCents * 0.05)}</p>` // pricing policy lives in markup
      : ""
  }
`;
```

### Do: presentation-only conditionals over prepared data

```typescript
const view = presentOrder(order, customer); // the model already computed any discount

const html = `
  <p>Total: ${view.totalFormatted}</p>
  ${view.discountFormatted ? `<p>Discount: ${view.discountFormatted}</p>` : ""}
`;
```


## Transform View

**Problem:** convert data into output in one transformation. **Use when:** output is structured and deterministic. **Tradeoff:** transformations can be harder to author incrementally and may mix data shaping with rendering. **Review:** ensure input is presentation-ready and output encoding is correct; use Template View when template-centric editing is the priority.


## Two Step View

**Problem:** apply consistent framing across many views. **Use when:** pages share layout while retaining distinct content rendering. **Tradeoff:** two render stages complicate data flow, escaping, and debugging. **Review:** ensure the first stage emits a clear logical page and the second only applies shared layout.


## Application Controller

**Problem:** coordinate navigation across related requests. **Use when:** multi-step flows share state and transition rules. **Tradeoff:** centralization can absorb domain policy and unrelated flows. **Review:** keep it to flow decisions, make transitions explicit, and leave business invariants in the domain.

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
