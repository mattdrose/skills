# Classes and Modules

A class or module should own a cohesive responsibility behind a small interface.

- Group code that changes for the same reason; separate code that changes independently.
- Keep public APIs smaller than implementations and expose only stable concepts.
- Put dependencies at construction or explicit call boundaries rather than fetching hidden globals.
- Prefer composition when inheritance does not model a genuine substitutable relationship.
- Ensure base abstractions do not depend on concrete derivatives.
- Keep configuration, state, and behavior with the layer that owns the decision.
- Avoid generic `Manager`, `Service`, and `Utils` containers that accumulate unrelated work.
- Delete pass-through layers that add no policy, translation, or isolation.

A module boundary earns its cost when it protects an invariant, clarifies ownership, or contains
change.

## Examples

### Don't: accumulate unrelated work behind a vague name

```typescript
// Registration, email delivery, reporting, and validation all change independently
class AccountManager {
  createAccount(email: string, password: string): Promise<Account> {
    /* ... */
  }

  sendWelcomeEmail(accountId: string): Promise<void> {
    /* ... */
  }

  exportAccountsToCsv(): Promise<string> {
    /* ... */
  }

  validatePasswordStrength(password: string): boolean {
    /* ... */
  }
}
```

### Do: one cohesive responsibility with explicit dependencies

```typescript
class AccountRegistration {
  private readonly accounts: AccountRepository;
  private readonly mailer: WelcomeMailer;

  constructor({ accounts, mailer }: { accounts: AccountRepository; mailer: WelcomeMailer }) {
    this.accounts = accounts;
    this.mailer = mailer;
  }

  async register({ email, password }: { email: string; password: string }): Promise<Account> {
    assertStrongPassword(password);

    const account = await this.accounts.create({ email, password });
    await this.mailer.sendWelcome(account);
    return account;
  }
}
```

### Don't: inherit just to reuse methods

```typescript
// A report is not a kind of database client; subclassing only grabs query()
class SalesReport extends PostgresClient {
  async build(): Promise<Report> {
    const rows = await this.query("SELECT * FROM sales");
    return summarize(rows);
  }
}
```

### Do: compose the capability you need

```typescript
class SalesReport {
  constructor(private readonly sales: SalesQueries) {}

  async build(): Promise<Report> {
    return summarize(await this.sales.recentSales());
  }
}
```
