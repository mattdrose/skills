# Web Presentation

Keep request handling, navigation, rendering, and domain policy distinct.

## Controllers

Use page controllers for isolated endpoints and a front controller for shared routing, authentication, and request setup.

### Don't: shared request setup copied into every page controller

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

### Do: a front controller for shared setup, page controllers for endpoints

```typescript
app.use(async (req, res, next) => {
  req.user = await verifyToken(req.headers.authorization);
  if (!req.user) return res.sendStatus(401);
  next();
});

app.get("/orders", async (req, res) => res.json(await listOrders(req.user)));
app.get("/invoices", async (req, res) => res.json(await listInvoices(req.user)));
```

## Views

Templates are direct and familiar. Transform views suit deterministic data-to-output conversion. Two-step views centralize a shared site layout.

### Don't: a template making domain decisions over raw input

```typescript
// unescaped query input, and refund policy is buried in markup
const html = `
  <h1>Order ${order.id}</h1>
  <p>${req.query.message}</p>
  ${daysSince(order.shippedAt) <= 30 ? `<a href="/refund">Request refund</a>` : ""}
`;
```

### Do: escape output and take decisions from the model

```typescript
const view = presentOrder(order); // the model decided refund eligibility

const html = `
  <h1>Order ${escapeHtml(view.orderId)}</h1>
  <p>${escapeHtml(String(req.query.message ?? ""))}</p>
  ${view.canRequestRefund ? `<a href="/refund">Request refund</a>` : ""}
`;
```

## MVC

The model owns application state and policy; the controller interprets input; the view renders. Framework names do not guarantee this separation.

## Review

Verify output encoding, authorization at the operation boundary, validation of untrusted input, idempotency where expected, and no domain decisions hidden in templates.
