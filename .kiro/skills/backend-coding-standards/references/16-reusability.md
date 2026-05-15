# Reusability, Maintainability & Scalability

## Hard rules (mapped to BE audit)

| Rule | Severity | Source |
|------|----------|--------|
| Within-lambda duplication (>2 files) | MAJOR | BE rule 39 |
| Cross-lambda duplication | not flagged | BE rule 39 |
| Single Responsibility violated (use case also handles logging/audit/infra) | MAJOR | BE checklist M |
| Unnecessary indirection layer (delegating class with no added logic) | MAJOR | BE rule 43 |
| Dual representation (interface AND class without clear boundary) | MAJOR | BE checklist M |
| Local interface duplicating domain model type | MAJOR | BE rule 53 |
| Adapter switching scattered across codebase | MAJOR | BE checklist M |
| Event schema not versioned (EventBridge / Kafka consumers) | MAJOR | BE checklist M |
| Feature flag hardcoded `if (env === 'prod')` instead of env-var flag | MAJOR | BE checklist M |
| Business constant hardcoded in source (not config or DB) | MAJOR | BE rule 41 |
| Commented-out business logic without ticket | MAJOR | BE rule 34 |
| Step Functions not used for >15 min workflows | MAJOR | BE checklist M |
| Dead factory/utility method (exported, never called) | MAJOR | BE rule 54 |

## SRP applied

```ts
// ❌ does too much
export class ProcessFmlaEventUseCase {
  async execute(event: FmlaEvent) {
    logger.info('start', { event });                              // logging
    await sendAuditEntry({ kind: 'fmla', payload: event });       // audit
    const validated = FmlaEventSchema.parse(event);                // validation
    const updated = await this.repo.upsertFmlaCase(validated);     // domain
    await this.kafka.publish('calm.fmla.event.v1', updated);       // integration
    logger.info('done', { id: updated.id });
    return updated;
  }
}

// ✅ split — orchestration only in the use case
export class ProcessFmlaEventUseCase {
  constructor(
    private readonly repo: LeaveRepositoryPort,
    private readonly publisher: WorkdayPublisherPort,
    private readonly audit: AuditPort,
  ) {}

  async execute(input: FmlaEvent): Promise<FmlaCase> {
    const result = await this.repo.upsertFmlaCase(input);
    await Promise.allSettled([
      this.publisher.publish(result),
      this.audit.record({ kind: 'fmla', payload: result }),
    ]);
    return result;
  }
}
```

The handler does logging + tracing. The use case orchestrates ports. The adapters carry I/O.

## Feature flags
- Boolean flags via env vars: `FEATURE_NEW_EVALUATOR=true`.
- Multi-value flags via SSM Parameter Store; cached at cold start, refreshed on TTL.
- Never `if (process.env.NODE_ENV === 'prod')` to gate behaviour.

## Constants
- Business constants in `domain/constants.ts` if behaviour-defining (e.g., FAA-117 lookback hours).
- Configuration constants in `infrastructure/config.ts`, sourced from env vars.
- No magic numbers in source — name every constant.

## Step Functions
- Workflows >15 min wall-clock time use Step Functions, not chained Lambdas.
- Long-poll patterns avoided (cost + cold-start amplification).
