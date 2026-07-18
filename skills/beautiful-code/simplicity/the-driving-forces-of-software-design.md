# Purpose and Value

Software exists to help people accomplish something. A change is useful only when it advances that purpose without imposing greater harm or cost.

## Review prompts

- What user or operator problem does this change solve?
- Is there evidence that the problem exists and matters?
- Does the behavior fit the product's stated purpose?
- Could an existing capability solve the problem already?
- Who benefits, how often, and by how much?
- What new burden does the change place on other users or maintainers?

Rank options by value relative to total effort:

- Include implementation, review, rollout, support, and future maintenance.
- Account for both likely benefit and the impact when the feature is useful.
- Prefer changes that can reach users soon enough to provide value.
- Treat speculative future value as uncertain, not guaranteed.

A cheap change with meaningful benefit often beats an ambitious change with marginal benefit. Over a system's lifetime, maintenance cost usually matters more than initial implementation cost.

### Don't: build new machinery when an existing capability solves the problem

```typescript
// Users asked to "get reports weekly"; scheduled email delivery already exists.
class ReportSchedulerService {
  constructor(
    private cron: CronRunner,
    private storage: BlobStore,
    private notifier: PushNotifier,
  ) {}

  async scheduleWeeklyExport(userId: string, format: "pdf" | "csv" | "xlsx"): Promise<void> {
    await this.cron.register(`report-${userId}`, "0 9 * * MON", async () => {
      const blob = await renderReport(userId, format);
      const url = await this.storage.upload(blob);
      await this.notifier.push(userId, `Your report is ready: ${url}`);
    });
  }
}
```

### Do: reuse the capability the product already has

```typescript
async function enableWeeklyReport(userId: string): Promise<void> {
  await emailSubscriptions.subscribe(userId, {
    digest: "weekly-report",
    day: "monday",
  });
}
```
