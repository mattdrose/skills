# Designing for Change

Long-lived software will change, but the exact changes are unknowable. Design from current evidence while keeping likely change paths local and safe.

## Prefer

- Small components with clear responsibilities.
- One authoritative location for each rule or fact.
- Explicit, narrow dependencies.
- Stable boundaries around volatile details and external systems.
- Incremental design driven by real features and defects.
- Decisions that are easy to reverse when uncertainty is high.

### Don't: copy one rule into places that must change together

```typescript
// The threshold is duplicated; the banner and the charge will drift apart.
function shippingBanner(cartTotal: number): string {
  return cartTotal >= 50 ? "Free shipping!" : "Add more for free shipping";
}

function shippingCost(cartTotal: number): number {
  return cartTotal >= 50 ? 0 : 7.99;
}
```

### Do: give the rule one authoritative location

```typescript
const FREE_SHIPPING_THRESHOLD = 50;

function qualifiesForFreeShipping(cartTotal: number): boolean {
  return cartTotal >= FREE_SHIPPING_THRESHOLD;
}

function shippingBanner(cartTotal: number): string {
  return qualifiesForFreeShipping(cartTotal) ? "Free shipping!" : "Add more for free shipping";
}

function shippingCost(cartTotal: number): number {
  return qualifiesForFreeShipping(cartTotal) ? 0 : 7.99;
}
```

## Avoid

- Features, extension points, or configuration without a current need.
- Generic frameworks built for imagined consumers.
- Rigid designs tied to one incidental workflow or representation.
- Copying knowledge into multiple places that must change together.
- Detailed plans that assume distant requirements are known.

Do not confuse adaptability with abstraction. Add a boundary when it isolates a demonstrated source of change; do not add one merely because something might vary someday. Unused flexibility has its own behavior, failure modes, and maintenance cost.

A strong design absorbs environmental change with a small, understandable software change.

### Don't: build a registry for one implementation

```typescript
// Only email exists today; the interface and registry serve imagined consumers.
interface NotificationChannel {
  send(recipient: string, body: string): Promise<void>;
}

class ChannelRegistry {
  private channels = new Map<string, NotificationChannel>();

  register(name: string, channel: NotificationChannel): void {
    this.channels.set(name, channel);
  }

  send(name: string, recipient: string, body: string): Promise<void> {
    const channel = this.channels.get(name);
    if (!channel) throw new Error(`Unknown channel: ${name}`);
    return channel.send(recipient, body);
  }
}
```

### Do: write the direct version; add the boundary when a second channel arrives

```typescript
async function sendOrderConfirmation(order: Order): Promise<void> {
  await emailClient.send({
    to: order.customerEmail,
    subject: `Order ${order.id} confirmed`,
    body: renderConfirmation(order),
  });
}
```
