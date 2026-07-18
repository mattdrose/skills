# Simple Code

Code is simple when its intended readers can understand its behavior and change it without tracing unnecessary concepts.

## Review prompts

- Does each piece have one clear responsibility?
- Is important control flow visible near the entry point?
- Are names precise without being cryptic or exhaustive?
- Are similar operations expressed consistently?
- Do modules expose a small interface and hide incidental detail?
- Do comments explain non-obvious reasons, constraints, or tradeoffs rather than restate code?
- Can dead code, indirection, configuration, or special cases be deleted?

Simplicity depends on audience and context. Familiarity alone does not make an interface simple; assess it from the perspective of a capable newcomer. Advanced language or platform features are justified only when they reduce the reader's total mental work.

Readability is necessary but not sufficient. Well-formatted code may still encode too many states, responsibilities, or interactions. Conversely, fewer lines may be harder to understand than a direct, slightly longer implementation.

### Don't: hide control flow behind clever compression

```typescript
// Fewer lines, but the reader must unpack reduce, spread, and a nested ternary at once.
const summary = orders.reduce(
  (acc, o) => ({
    ...acc,
    [o.status]: (acc[o.status] ?? 0) + (o.status === "refunded" ? -o.total : o.total),
  }),
  {} as Record<string, number>,
);
```

### Do: write the direct, slightly longer version

```typescript
const summary: Record<string, number> = {};
for (const order of orders) {
  const amount = order.status === "refunded" ? -order.total : order.total;
  summary[order.status] = (summary[order.status] ?? 0) + amount;
}
```

### Don't: mix responsibilities so no piece can be understood alone

```typescript
async function registerUser(input: unknown): Promise<void> {
  const data = input as { email?: string; password?: string };
  if (!data.email || !data.email.includes("@")) throw new Error("bad email");
  if (!data.password || data.password.length < 12) throw new Error("bad password");
  const hash = await argon2.hash(data.password);
  await db.insert("users", { email: data.email.toLowerCase(), hash });
  await mailer.send(data.email, "Welcome!", renderWelcome(data.email));
  metrics.increment("signup");
}
```

### Do: give each piece one clear responsibility

```typescript
async function registerUser(input: unknown): Promise<void> {
  const { email, password } = validateRegistration(input);
  const user = await createUser(email, password);
  await sendWelcomeEmail(user);
  metrics.increment("signup");
}
```

### Don't: keep indirection that only forwards

```typescript
// Each layer just delegates; the reader traces all of them to find any logic.
class UserService {
  constructor(private manager: UserManager) {}
  getUser(id: string): Promise<User> {
    return this.manager.getUser(id);
  }
}

class UserManager {
  constructor(private repo: UserRepository) {}
  getUser(id: string): Promise<User> {
    return this.repo.findById(id);
  }
}
```

### Do: delete pass-through layers

```typescript
class UserService {
  constructor(private repo: UserRepository) {}
  getUser(id: string): Promise<User> {
    return this.repo.findById(id);
  }
}
```
