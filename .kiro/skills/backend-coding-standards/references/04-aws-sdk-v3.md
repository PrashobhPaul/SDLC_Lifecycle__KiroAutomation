# AWS SDK v3 & DI Container

## Required: SDK v3 modular imports only (BE rule 14)

```ts
// ✅ correct
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, GetCommand, QueryCommand, UpdateCommand } from '@aws-sdk/lib-dynamodb';
import { SQSClient, SendMessageCommand } from '@aws-sdk/client-sqs';

// ❌ wrong — SDK v2
import * as AWS from 'aws-sdk';
```

## DI container — singletons + lazy-init

```ts
// infrastructure/aws-clients.ts
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb';

let _ddb: DynamoDBDocumentClient | undefined;

export function getDynamoDb(): DynamoDBDocumentClient {
  if (!_ddb) {
    const base = new DynamoDBClient({
      region: process.env.AWS_REGION,
      maxAttempts: 3,
      requestHandler: { connectionTimeout: 3000, requestTimeout: 5000 },
    });
    _ddb = DynamoDBDocumentClient.from(base, {
      marshallOptions: { removeUndefinedValues: true, convertClassInstanceToMap: false },
    });
  }
  return _ddb;
}
```

```ts
// infrastructure/di-container.ts
import { DdbPilotRepository } from '../adapters/outbound/ddb-pilot.repository';
import { GetPilotByIdUseCase } from '../application/get-pilot-by-id.usecase';

const registry = new Map<string, unknown>();

export const container = {
  resolve<T>(name: string): T {
    if (!registry.has(name)) registry.set(name, build(name));
    return registry.get(name) as T;
  },
};

function build(name: string): unknown {
  switch (name) {
    case 'pilotRepository':    return new DdbPilotRepository();
    case 'getPilotByIdUseCase': return new GetPilotByIdUseCase(container.resolve('pilotRepository'));
    default: throw new Error(`Unknown registration: ${name}`);
  }
}
```

## Hard rules (mapped to BE audit)

| Rule | Severity | Source |
|------|----------|--------|
| SDK v2 anywhere | MAJOR per file | BE rule 14 |
| SDK client instantiated inside handler body or catch block | MAJOR | BE rule 29 |
| Singleton client mutated across requests (BE shared mutable state) | MAJOR | BE rule 39 |
| Module-level mutable state without lazy-init | MAJOR | BE checklist G |
| Adapter switching not isolated to factory/container | MAJOR | BE checklist M |

## Retry & timeout
- Set `maxAttempts: 3` on every client.
- Set explicit `connectionTimeout` and `requestTimeout`. Defaults are too long for Lambda.
- Use AWS SDK middleware for X-Ray tracing: `captureAWSv3Client(ddbBase)` or OTEL instrumentation.

## Connection reuse
- `AWS_NODEJS_CONNECTION_REUSE_ENABLED=1` in Lambda env vars (set in Terraform Lambda defaults).
- Module-level client reuse across invocations.
