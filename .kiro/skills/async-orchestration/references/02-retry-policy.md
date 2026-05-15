# Retry Policy

## Defaults

```ts
const RETRY_POLICY = {
  max_attempts: 3,
  initial_delay_ms: 1000,
  multiplier: 4,
  jitter_pct: 0.2,
  retryable_errors: [
    'ETIMEDOUT', 'ECONNRESET', 'ENOTFOUND', 'EAI_AGAIN',
    'RequestTimeout', 'ServiceUnavailable', 'ThrottlingException',
    'ProvisionedThroughputExceededException', '5xx'
  ],
  non_retryable_errors: [
    'ValidationError', '4xx', 'AccessDenied', 'IAMException',
    'SchemaValidationError', 'IdempotencyKeyConflict'
  ],
};

function nextDelayMs(attempt: number): number {
  const base = RETRY_POLICY.initial_delay_ms * Math.pow(RETRY_POLICY.multiplier, attempt - 1);
  const jitter = base * RETRY_POLICY.jitter_pct * (Math.random() * 2 - 1);
  return Math.max(0, Math.floor(base + jitter));
}
```

Schedule: ~1s → ~4s → ~16s (jittered ±20%).

## Classification
- Retryable: transient network or AWS throttling.
- Non-retryable: 4xx authorization, schema validation, idempotency conflict.

The classifier is a typed helper:

```ts
function classifyError(err: unknown): 'retryable' | 'non-retryable' {
  if (!(err instanceof Error)) return 'non-retryable';
  const msg = err.message + ' ' + ((err as any).code ?? '');
  if (RETRY_POLICY.non_retryable_errors.some((p) => msg.includes(p))) return 'non-retryable';
  if (RETRY_POLICY.retryable_errors.some((p) => msg.includes(p))) return 'retryable';
  return 'non-retryable';
}
```

## After max_attempts
- Route to DLQ.
- Trigger fallback path.
- Emit AIQ record with `async.dlq_routed=true` and `async.fallback_used=true`.
- Operations dashboard alerts `kiro_dlq_depth > 0`.
