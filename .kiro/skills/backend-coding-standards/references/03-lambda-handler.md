# Lambda Handler Shape

Thin handler. Parse event → call use case → return response. No business logic in the handler.

## AppSync resolver Lambda

```ts
import type { AppSyncResolverEvent, AppSyncIdentityCognito } from 'aws-lambda';
import { Logger } from '@aws-lambda-powertools/logger';
import { Tracer } from '@aws-lambda-powertools/tracer';
import { container } from '../infrastructure/di-container';
import { GetPilotByIdInputSchema } from '../application/get-pilot-by-id.schema';

const logger = new Logger({ serviceName: 'pilot-resolver' });
const tracer = new Tracer({ serviceName: 'pilot-resolver' });
const useCase = container.resolve('getPilotByIdUseCase');

type Args = { pilotId: string };
type Identity = AppSyncIdentityCognito & { resolverContext: { adGroups: string[]; roles: string[] } };

export const handler = tracer.captureLambdaHandler(
  async (event: AppSyncResolverEvent<Args, unknown, Identity>): Promise<unknown> => {
    const parsed = GetPilotByIdInputSchema.parse(event.arguments);
    logger.appendKeys({ pilotId: parsed.pilotId });
    return useCase.execute(parsed, {
      adGroups: event.identity?.resolverContext?.adGroups ?? [],
      roles:    event.identity?.resolverContext?.roles ?? [],
    });
  }
);
```

## SQS consumer Lambda

```ts
import type { SQSBatchResponse, SQSEvent, SQSRecord } from 'aws-lambda';

const useCase = container.resolve('processFmlaEventUseCase');

export const handler = tracer.captureLambdaHandler(
  async (event: SQSEvent): Promise<SQSBatchResponse> => {
    const failed: { itemIdentifier: string }[] = [];
    await Promise.allSettled(
      event.Records.map(async (record: SQSRecord) => {
        try {
          const parsed = FmlaEventSchema.parse(JSON.parse(record.body));
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

| Rule | Severity | Source |
|------|----------|--------|
| Handler thin (no business logic) | MAJOR per instance | BE checklist D |
| Event source typed correctly | MAJOR per handler | BE rule 47 |
| Input validated at handler boundary only | MAJOR if duplicated in use case | BE rule 51 |
| IDs/timestamps generated at one layer only | MAJOR if generated then ignored downstream | BE rule 52 |
| Correct response shape per event source | MAJOR | BE checklist D |
| SDK clients/use cases at module level (not in handler/catch) | MAJOR | BE rule 29 |
| `context.callbackWaitsForEmptyEventLoop = false` if open connections | MAJOR | BE checklist D |
| Handler does not swallow errors that should propagate | MAJOR | BE checklist D |
| `batchItemFailures` returned for SQS | MAJOR | BE rule 18 |
