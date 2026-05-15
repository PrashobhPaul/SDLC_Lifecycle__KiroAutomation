# Consumer Patterns (SQS · SNS · EventBridge)

## Required pattern for every consumer

```ts
// === handler/consumer.ts (inbound adapter) ===

import type { SQSBatchResponse, SQSEvent, SQSRecord } from 'aws-lambda';
import { Logger } from '@aws-lambda-powertools/logger';
import { Tracer } from '@aws-lambda-powertools/tracer';
import { container } from '../infrastructure/di-container';

const logger = new Logger({ serviceName: 'fmla-consumer' });
const tracer = new Tracer({ serviceName: 'fmla-consumer' });
const useCase = container.resolve('processFmlaEventUseCase');

export const handler = tracer.captureLambdaHandler(
  async (event: SQSEvent): Promise<SQSBatchResponse> => {
    const failed: { itemIdentifier: string }[] = [];

    await Promise.allSettled(
      event.Records.map(async (record: SQSRecord) => {
        try {
          const parsed = parseAndValidate(record.body);
          await useCase.execute(parsed, { idempotencyKey: parsed.event_id });
        } catch (err) {
          logger.error('record-failed', { messageId: record.messageId, err });
          failed.push({ itemIdentifier: record.messageId });
        }
      })
    );

    return { batchItemFailures: failed };
  }
);
```

## Required behaviours

### Idempotency
- `event_id` (or DMS sequence number) is the idempotency key.
- The use case checks the idempotency table BEFORE applying the side effect.
- Successful processing records the key for the configured TTL (>= 7 days for CDC events).

### batchItemFailures
- SQS consumers always return `{ batchItemFailures: [...] }` (BE rule 18).
- Failed items are returned to the queue; succeeded items are removed.
- Without this, partial-batch failure causes the entire batch to retry, amplifying load.

### DLQ wiring
- SQS source queues have a DLQ with `maxReceiveCount=5` (or appropriate per use case).
- SNS topics have a redrive policy to a DLQ on the subscriber side.
- EventBridge rules have a target DLQ via `dead_letter_config`.

### Retry classification
- Transient errors (5xx from downstream, throttling) → throw to retry.
- Permanent errors (validation, auth) → log, mark as success on the queue, write to a separate "poison" queue for inspection.

### Concurrency
- Reserved concurrency on the consumer Lambda where ordering or downstream rate-limits demand it.
- For SQS FIFO consumers, concurrency = number of message groups.

### Tracing
- X-Ray on; trace context propagated via record attributes (`AWSTraceHeader` for SQS, `_X_AMZN_TRACE_ID` for EventBridge).
- Span attributes: `event_id`, `schema_version`, `event_source`.

## Anti-patterns to flag (Code Reviewer)
- Consumer with no idempotency check.
- Consumer that swallows errors silently (`catch {}`).
- Consumer doing business logic in the handler instead of a use case.
- Consumer instantiating SDK clients inside the handler body (BE rule 29).
- `await` inside a `for` loop where parallel processing is safe.
- `Promise.all` where partial failure must be handled (`Promise.allSettled` is required).
