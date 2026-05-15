# Review — Architectural Notes (Code Reviewer Output)

## Layering

- Hexagonal compliance: <pass / fail with notes>
- Cross-feature import audit: <pass / fail with notes>

## Module boundaries

- Domain → adapter direction respected: <yes/no>
- `@aws-sdk` imports in domain layer: <count, must be 0>

## Multi-region posture

- Region/account/ARN hardcoding: <count, must be 0>
- IAM least-privilege cross-ref: <pass/fail>

## GraphQL

- Breaking changes flagged with `consider-usage`: <yes/no>
- N+1 risks identified: <list with DataLoader recommendations>

## Async

- DLQ on every async-trigger Lambda: <pass/fail>
- Idempotency-key on every SQS/SNS/EventBridge consumer: <pass/fail>
- `batchItemFailures` returned by every SQS consumer: <pass/fail>

## Test architecture

- Coverage thresholds met per layer: <yes/no>
- Test-quality red flags (snapshot-only, mock-data type drift): <list>
