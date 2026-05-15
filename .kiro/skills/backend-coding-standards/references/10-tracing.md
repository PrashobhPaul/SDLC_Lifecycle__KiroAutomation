# OTEL Tracing

## Powertools Tracer (X-Ray-compatible)

```ts
import { Tracer } from '@aws-lambda-powertools/tracer';

export const tracer = new Tracer({
  serviceName: process.env.POWERTOOLS_SERVICE_NAME ?? 'unnamed',
  captureHTTPsRequests: true,
});

const ddbClient = tracer.captureAWSv3Client(getDynamoDb());
```

## Required behaviours

| Rule | Severity | Source |
|------|----------|--------|
| Tracing enabled (X-Ray or OTEL) | MAJOR if absent | BE rule 17 |
| Trace IDs in logs | MAJOR if missing | BE checklist F |
| Spans named per operation | WARN if generic | best practice |
| AWS SDK calls captured automatically | MAJOR if instrumentation missing | BE checklist F |

## Subsegments per use case

```ts
export class GetPilotByIdUseCase {
  async execute(input: GetPilotByIdInput): Promise<Pilot> {
    return tracer.provider.getActiveSpan()?.startActiveSpan('use-case.get-pilot-by-id', async (span) => {
      span.setAttribute('pilotId', input.pilotId);
      const pilot = await this.repo.findById(input.pilotId);
      if (!pilot) throw new NotFoundError(`Pilot ${input.pilotId} not found`);
      return pilot;
    }) as Pilot;
  }
}
```

## Cross-system propagation
- `AWSTraceHeader` from SQS records → continue the trace.
- `_X_AMZN_TRACE_ID` from EventBridge → continue the trace.
- Outbound HTTP requests → propagate the trace header automatically when using SDK-instrumented clients.
- Kafka: producer attaches headers (`b3`, `traceparent`); consumer extracts.

## Backend
- Phase 1: AWS X-Ray.
- Phase 2 (post-pilot): OTEL Collector → Tempo / Jaeger; X-Ray remains for AWS-native correlation.
