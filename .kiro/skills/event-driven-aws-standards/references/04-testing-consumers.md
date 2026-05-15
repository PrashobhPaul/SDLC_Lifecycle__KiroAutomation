# Testing Event Consumers

## Test layers

### Unit (use case)
- No AWS SDK. Mock the ports.
- Cover: happy path, validation failure, downstream failure, idempotency-replay, schema-version mismatch.

### Adapter
- aws-sdk-client-mock for SQS / Kinesis / DynamoDB.
- Cover: partial-batch failure returns correct `batchItemFailures`.
- Cover: tracing context propagated from `AWSTraceHeader` / record attributes.

### Integration
- Testcontainers: Localstack for SQS/SNS/EventBridge/DynamoDB; Redpanda or Kafka container for Kafka.
- Test fixtures: send 10 messages, assert side effects, assert idempotency replay.
- Cover: DLQ routing after `maxReceiveCount`.

## Required tests per consumer

| # | Name | What it asserts |
|---|------|-----------------|
| 1 | happy-path | One record in, one side effect, one ack |
| 2 | partial-batch | 5 records, 2 fail; failures returned in `batchItemFailures` |
| 3 | idempotency-replay | Same `event_id` twice; only one side effect |
| 4 | schema-version-newer | v2 record on v1 consumer; rejected, no side effect |
| 5 | schema-version-older | v1 record on v2 consumer; processed via the v1 path |
| 6 | downstream-throttled | Downstream throws ThrottlingException; record retried |
| 7 | downstream-permanent | Downstream throws ValidationError; record sent to poison queue |
| 8 | tracing-propagation | Trace attributes captured in spans |

## Coverage targets per layer
- Use case: ≥ 90%
- Adapter: ≥ 80%
- Integration: as many flows as the use case has decision branches

## Determinism
- Test data factories are seeded; no Math.random in tests.
- Time-of-day in fixtures is fixed via `vi.useFakeTimers()` / `jest.useFakeTimers()`.
