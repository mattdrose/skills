# Code Smells

Smells are prompts to investigate, not proof that code is wrong. Judge them in context and identify the change pressure they create.

| Signal                     | Risk                                 | Typical response                                 |
| -------------------------- | ------------------------------------ | ------------------------------------------------ |
| Duplicated knowledge       | fixes diverge                        | centralize the rule, not merely identical syntax |
| Long function              | mixed levels of abstraction          | extract named steps or simplify the algorithm    |
| Large module/class         | unrelated reasons to change          | split by responsibility                          |
| Long parameter list        | hidden concept or excessive coupling | pass a meaningful object or move behavior        |
| Divergent change           | one unit serves unrelated concerns   | separate responsibilities                        |
| Shotgun surgery            | one rule is scattered                | move ownership to one boundary                   |
| Feature envy               | behavior lives away from its data    | move behavior toward the data owner              |
| Data clumps                | values travel together repeatedly    | introduce a domain value                         |
| Primitive obsession        | invalid states are easy to represent | use a constrained domain type                    |
| Repeated branching by type | behavior varies by kind              | use polymorphism or a strategy when stable       |
| Temporary field/state      | lifecycle is unclear                 | split the lifecycle or extract an object         |
| Message chain              | callers know internal navigation     | expose an intention-level operation              |
| Middleman                  | delegation adds no boundary value    | remove the layer                                 |
| Inappropriate intimacy     | internals leak across modules        | narrow the interface or move responsibility      |
| Refused inheritance        | subtype violates parent expectations | replace inheritance with composition             |
| Speculative generality     | abstractions have no current use     | inline or delete them                            |

## Review questions

1. What concrete maintenance problem does the smell cause?
2. Is the problem local or evidence of misplaced ownership?
3. What is the smallest transformation that removes it?
4. Does the proposed design make the next likely change easier?

## Examples

### Don't: duplicate one business rule in two places

```typescript
function invoiceTotal(lines: Line[]): number {
  const subtotal = lines.reduce((sum, l) => sum + l.price * l.qty, 0);
  return subtotal > 1000 ? subtotal * 0.95 : subtotal;
}

function quoteTotal(lines: Line[]): number {
  const subtotal = lines.reduce((sum, l) => sum + l.price * l.qty, 0);
  return subtotal > 1000 ? subtotal * 0.95 : subtotal; // a fix here will miss the copy above
}
```

### Do: centralize the rule

```typescript
const subtotal = (lines: Line[]) => lines.reduce((sum, l) => sum + l.price * l.qty, 0);

const applyVolumeDiscount = (amount: number) => (amount > 1000 ? amount * 0.95 : amount);

const invoiceTotal = (lines: Line[]) => applyVolumeDiscount(subtotal(lines));
const quoteTotal = (lines: Line[]) => applyVolumeDiscount(subtotal(lines));
```

### Don't: let a data clump travel as loose primitives

```typescript
function shipOrder(order: Order, street: string, city: string, postal: string, country: string) {}
function validateAddress(street: string, city: string, postal: string, country: string) {}
function addressLabel(street: string, city: string, postal: string, country: string) {}
```

### Do: introduce a domain value

```typescript
interface Address {
  street: string;
  city: string;
  postal: string;
  country: string;
}

function shipOrder({ order, destination }: { order: Order; destination: Address }) {}
function validateAddress(address: Address) {}
function addressLabel(address: Address) {}
```

### Don't: compute another object's answers from its data

```typescript
class OrderReport {
  format(order: Order): string {
    // this class knows Order's internals better than Order does
    const total = order.lines.reduce((s, l) => s + l.price * l.qty, 0);
    const withTax = total * (1 + order.customer.region.taxRate);
    return `${order.id}: ${withTax.toFixed(2)}`;
  }
}
```

### Do: move behavior toward the data owner

```typescript
class Order {
  get total(): number {
    return this.lines.reduce((s, l) => s + l.price * l.qty, 0);
  }
  get totalWithTax(): number {
    return this.total * (1 + this.customer.region.taxRate);
  }
}

class OrderReport {
  format(order: Order): string {
    return `${order.id}: ${order.totalWithTax.toFixed(2)}`;
  }
}
```
