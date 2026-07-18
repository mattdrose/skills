# Simplifying Conditionals

Conditionals should reveal business decisions rather than obscure them with mechanics.

## Transformations

- **Extract condition:** name a non-obvious predicate or branch action.
- **Consolidate conditions:** combine checks that lead to the same result when they express one rule.
- **Deduplicate branches:** move identical setup or cleanup outside the conditional.
- **Use guard clauses:** handle exceptional or terminal cases early so the main path remains visible.
- **Remove control flags:** return, break, or extract a function instead of mutating a flag that simulates control flow.
- **Replace repeated type branching:** move stable variant behavior behind polymorphism or a strategy.
- **Model absence explicitly:** use an option/result type or null object only when its semantics are uniform and unsurprising.
- **Assert internal invariants:** reserve assertions for programmer errors, not untrusted input.

### Don't: bury the main path in nested conditionals

```typescript
function payAmount(employee: Employee): number {
  let result: number;
  if (employee.isSeparated) {
    result = 0;
  } else {
    if (employee.isRetired) {
      result = employee.pension;
    } else {
      result = employee.baseSalary + employee.overtime; // the normal case, three levels deep
    }
  }
  return result;
}
```

### Do: use guard clauses for terminal cases

```typescript
function payAmount(employee: Employee): number {
  if (employee.isSeparated) return 0;
  if (employee.isRetired) return employee.pension;
  return employee.baseSalary + employee.overtime;
}
```

### Don't: simulate control flow with a mutable flag

```typescript
function findMiscreant(people: string[]): string {
  let found = ""; // control flag mutated to steer the loop
  for (const person of people) {
    if (!found) {
      if (person === "Don" || person === "John") {
        found = person;
      }
    }
  }
  return found;
}
```

### Do: return directly instead of mutating a flag

```typescript
function findMiscreant(people: string[]): string {
  for (const person of people) {
    if (person === "Don" || person === "John") return person;
  }
  return "";
}
```

### Don't: repeat the same type switch at every call site

```typescript
function speedOf(bird: Bird): number {
  switch (bird.type) {
    case "european":
      return 35;
    case "african":
      return 40 - 2 * bird.coconuts;
    case "norwegian":
      return bird.isNailed ? 0 : 10 * bird.voltage; // every new bird edits every switch
  }
}
```

### Do: move variant behavior behind polymorphism

```typescript
class EuropeanSwallow {
  get speed(): number {
    return 35;
  }
}

class AfricanSwallow {
  constructor(private coconuts: number) {}
  get speed(): number {
    return 40 - 2 * this.coconuts;
  }
}
```

## Cautions

Do not merge conditions that happen to share an outcome but represent different policies likely to evolve independently. Polymorphism is not automatically simpler than a small, local switch. Guard clauses should clarify precedence rather than conceal required cleanup or partial mutation.

## Review checks

Verify branch order, short-circuit behavior, side effects, exception behavior, and exhaustive handling. Tests should cover meaningful decision boundaries, not every syntactic path in isolation.
