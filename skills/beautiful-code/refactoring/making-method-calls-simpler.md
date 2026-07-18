# Improving Interfaces

An interface should express intent, minimize knowledge, and make misuse difficult.

## Transformations

- Rename operations to match domain language and observable effect.
- Remove unused parameters; add parameters only when the caller should choose the value.
- Separate queries from commands when hidden mutation makes use surprising.
- Replace boolean or mode parameters with explicit operations when they select distinct behaviors.
- Preserve a whole domain object instead of passing several of its fields, unless that increases coupling unnecessarily.
- Introduce a parameter object when a recurring group has shared meaning or validation.
- Replace a parameter with an internal query only when the dependency properly belongs to the receiver and remains testable.
- Hide setters and methods that should not be part of the supported contract.
- Use factories when construction requires policy, validation, caching, or subtype selection.
- Convert error codes to typed failures or exceptions according to the surrounding contract.
- Check expected preconditions before an operation rather than using exceptions as ordinary branching.

### Don't: select distinct behaviors with a boolean flag

```typescript
function setDelivery(order: Order, isRush: boolean): void {
  if (isRush) {
    order.deliveryDate = addDays(order.placedAt, 1);
    order.fee = 15;
  } else {
    order.deliveryDate = addDays(order.placedAt, 5);
    order.fee = 0;
  }
}

setDelivery(order, true); // caller can't tell what true means
```

### Do: expose explicit operations

```typescript
function setRushDelivery(order: Order): void {
  order.deliveryDate = addDays(order.placedAt, 1);
  order.fee = 15;
}

function setStandardDelivery(order: Order): void {
  order.deliveryDate = addDays(order.placedAt, 5);
  order.fee = 0;
}

setRushDelivery(order);
```

### Don't: hide mutation inside a query

```typescript
class Account {
  getBalance(): number {
    this.lastAccessed = new Date(); // caller asking a question changes state
    this.accessCount++;
    return this.balance;
  }
}
```

### Do: separate the query from the command

```typescript
class Account {
  getBalance(): number {
    return this.balance;
  }

  recordAccess(): void {
    this.lastAccessed = new Date();
    this.accessCount++;
  }
}
```

## Compatibility

Public API changes require migration planning. Add the new entry point, migrate callers, deprecate the old one, and remove it only when compatibility permits. Preserve error semantics, defaults, nullability, and evaluation order unless intentionally changing behavior.

## Review checks

Can a caller understand the operation without reading its body? Are required dependencies explicit? Can invalid combinations be represented? Does convenience hide I/O, mutation, or expensive work? Is the interface narrower after the change?
