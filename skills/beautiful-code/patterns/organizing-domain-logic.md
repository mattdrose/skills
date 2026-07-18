# Organizing Domain Logic

Choose the simplest model that keeps business rules coherent.

## Transaction Script

Good for a small set of simple operations. Move shared or interacting rules into richer domain types before scripts become duplicated workflows.

## Table Module

Fits record-oriented data where one object coordinates logic for a table or result set.

## Domain Model

Fits behavior-rich domains with identities, relationships, and rules that interact across use cases.

## Service Layer

Defines application operations and transaction boundaries. It coordinates domain behavior; it should not become the only place business rules live.

### Don't: every rule piles into the service layer

```typescript
class SubscriptionService {
  async renew(id: string): Promise<void> {
    const sub = await this.repo.byId(id);
    if (sub.status === "canceled") throw new Error("cannot renew");
    if (sub.plan === "annual") sub.expiresAt = addYears(sub.expiresAt, 1);
    else sub.expiresAt = addMonths(sub.expiresAt, 1);
    sub.renewalCount += 1; // domain rules stranded outside the domain type
    await this.repo.save(sub);
  }
}
```

### Do: the service coordinates; the domain type owns the rules

```typescript
class SubscriptionService {
  async renew(id: string): Promise<void> {
    const sub = await this.repo.byId(id);
    sub.renew();
    await this.repo.save(sub);
  }
}

class Subscription {
  renew(): void {
    if (this.status === "canceled") throw new CannotRenewError(this.id);
    this.expiresAt = this.plan.extend(this.expiresAt);
    this.renewalCount += 1;
  }
}
```
