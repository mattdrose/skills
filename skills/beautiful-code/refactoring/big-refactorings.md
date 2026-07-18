# Large-Scale Refactoring

Architectural refactoring is a direction sustained by many small, releasable changes—not a long-lived rewrite branch.

## Common directions

- **Separate tangled variation:** when one hierarchy represents multiple independent dimensions, retain one and move the other behind composition.
- **Move from procedures to domain ownership:** identify records and the procedures that manipulate them, then move one coherent rule at a time to an owning object or module.
- **Separate domain from presentation:** keep business rules independent of UI, transport, and formatting concerns; introduce explicit boundary models where useful.
- **Split an overloaded type:** group fields and behavior by lifecycle or variation, then extract collaborators or variants incrementally.

### Don't: mix domain rules into presentation

```typescript
class InvoicePage {
  render(invoice: Invoice): string {
    // pricing policy is trapped inside the UI layer
    let total = invoice.lines.reduce((s, l) => s + l.price * l.qty, 0);
    if (invoice.customer.tier === "gold") total *= 0.9;
    if (total > 500) total -= 25;
    return `<h1>Invoice ${invoice.id}</h1><p>Due: ${total.toFixed(2)}</p>`;
  }
}
```

### Do: separate domain from presentation

```typescript
class InvoicePricing {
  amountDue(invoice: Invoice): number {
    let total = invoice.lines.reduce((s, l) => s + l.price * l.qty, 0);
    if (invoice.customer.tier === "gold") total *= 0.9;
    return total > 500 ? total - 25 : total;
  }
}

class InvoicePage {
  constructor(private pricing: InvoicePricing) {}
  render(invoice: Invoice): string {
    return `<h1>Invoice ${invoice.id}</h1><p>Due: ${this.pricing.amountDue(invoice).toFixed(2)}</p>`;
  }
}
```

### Don't: encode two independent dimensions in one hierarchy

```typescript
class Report {}
class PdfSalesReport extends Report {}
class PdfInventoryReport extends Report {}
class CsvSalesReport extends Report {}
class CsvInventoryReport extends Report {} // every new format multiplies every report type
```

### Do: retain one dimension and compose the other

```typescript
interface Format {
  render(data: ReportData): string;
}

class SalesReport {
  constructor(private format: Format) {}
  publish(): string {
    return this.format.render(this.collectData());
  }
}
```

## Execution strategy

1. State the concrete capability or maintenance problem driving the work.
2. Identify seams, dependencies, and observable contracts.
3. Add characterization and boundary coverage.
4. Define the target direction without designing every intermediate detail.
5. Migrate one path at a time, using temporary adapters when necessary.
6. Keep both paths consistent only for a bounded transition.
7. Remove obsolete code and compatibility layers promptly.
8. Reassess after each delivered improvement.

## Review risks

Watch for dual sources of truth, cycles, leaky adapters, expanding scope, hidden data migrations, and branches that cannot ship independently. Require a rollback or containment strategy for changes to persistence, protocols, or critical runtime paths.

Success is measured by easier product changes and clearer ownership, not by conformity to a diagram.
