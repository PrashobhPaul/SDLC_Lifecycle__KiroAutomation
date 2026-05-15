# Structured Logging

## Powertools logger (preferred)

```ts
import { Logger } from '@aws-lambda-powertools/logger';

export const logger = new Logger({
  serviceName: process.env.POWERTOOLS_SERVICE_NAME ?? 'unnamed',
  logLevel: (process.env.LOG_LEVEL as 'DEBUG' | 'INFO' | 'WARN' | 'ERROR') ?? 'INFO',
  persistentLogAttributes: {
    awsRegion: process.env.AWS_REGION,
    boundedContext: process.env.BOUNDED_CONTEXT,
  },
});

logger.info('processing-fmla-event', {
  fmlaCaseId: parsed.fmlaCase.id,
  schemaVersion: parsed.schema_version,
});
```

## Hard rules

| Rule | Severity | Source |
|------|----------|--------|
| `console.log` outside the logger | MAJOR per file | BE rule 15 |
| Logger inconsistent across files | MAJOR | BE checklist F |
| Log payload contains PII (names, IDs, DOB, emails) | MAJOR per field | BE checklist F |
| Log payload contains secret/token | CRITICAL | BE checklist F |
| Log message free-form string instead of structured object | MAJOR | BE checklist F |
| Log level hardcoded (not from env) | MAJOR | BE checklist F |
| CloudWatch log group missing retention | MAJOR | BE rule 46 |

## Required attributes
- Correlation/trace IDs (`logger.appendKeys({ traceId, spanId })`).
- Operation name.
- Business context (without PII).
- AWS request id (Powertools auto-injects this from context).

## Levels
- `DEBUG` — local + dev1 namespace only; should never reach prod.
- `INFO` — high-level milestones.
- `WARN` — recoverable issue with user-visible impact.
- `ERROR` — failure or unexpected state.

## PII redaction
- Use `tools/redactor.ts` to redact before logging.
- For Powertools, configure `logger.removeKeys` for known sensitive fields and process them upstream.

## Sample at high volume
- For per-record consumer logs, sample at INFO; log every error.
- Per-handler counter metrics (CloudWatch EMF) preferred over per-call info logs at scale.
