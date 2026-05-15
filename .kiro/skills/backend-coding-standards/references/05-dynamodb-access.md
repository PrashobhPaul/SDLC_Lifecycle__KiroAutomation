# DynamoDB Access Patterns

## Single-item access — repository pattern

```ts
// adapters/outbound/ddb-pilot.repository.ts
import { GetCommand, PutCommand, UpdateCommand, QueryCommand } from '@aws-sdk/lib-dynamodb';
import type { PilotRepositoryPort } from '../../ports/outbound/pilot-repository.port';
import type { Pilot } from '../../domain/pilot';
import { getDynamoDb } from '../../infrastructure/aws-clients';
import { config } from '../../infrastructure/config';

export class DdbPilotRepository implements PilotRepositoryPort {
  private readonly ddb = getDynamoDb();
  private readonly table = config.tables.pilots;

  async findById(pilotId: string): Promise<Pilot | null> {
    const out = await this.ddb.send(new GetCommand({
      TableName: this.table,
      Key: { pk: `PILOT#${pilotId}`, sk: 'PROFILE' },
      ProjectionExpression: 'pilotId, #n, base, updatedAt',
      ExpressionAttributeNames: { '#n': 'name' },
    }));
    return out.Item ? this.toDomain(out.Item) : null;
  }

  async listByBase(base: string, limit: number, cursor?: string): Promise<{ items: Pilot[]; nextCursor?: string }> {
    const out = await this.ddb.send(new QueryCommand({
      TableName: this.table,
      IndexName: 'byBase',
      KeyConditionExpression: '#b = :b',
      ExpressionAttributeNames: { '#b': 'base' },
      ExpressionAttributeValues: { ':b': base },
      Limit: limit,
      ExclusiveStartKey: cursor ? JSON.parse(Buffer.from(cursor, 'base64').toString()) : undefined,
    }));
    return {
      items: (out.Items ?? []).map((it) => this.toDomain(it)),
      nextCursor: out.LastEvaluatedKey
        ? Buffer.from(JSON.stringify(out.LastEvaluatedKey)).toString('base64')
        : undefined,
    };
  }

  private toDomain(item: Record<string, unknown>): Pilot {
    // map raw DDB shape into the domain object
    // adapter is the only place this leak happens
  }
}
```

## Hard rules

| Rule | Severity | Source |
|------|----------|--------|
| Full-table `Scan` without justification | MAJOR | BE rule 27 |
| Missing `ProjectionExpression` when subset of attrs needed | MAJOR | BE checklist G |
| GSI scan fallback (no key-condition) | MAJOR | BE checklist G |
| N+1 (query list → one call per item) | MAJOR | BE rule 33 |
| In-memory pagination (`.slice()` on full result) | MAJOR | BE rule 32 |
| `Promise.all` where partial failure must be handled | MAJOR | BE rule 28 |
| Missing batchWrite/transactWrite for multi-item mutations | MAJOR | BE checklist G |
| `ConsistentRead` not documented when used | WARN | BE checklist G |
| Hot partition key (e.g., date-only) | MAJOR | BE checklist G |
| Raw SDK shape leaking into domain | MAJOR | BE rule 70 |

## Pagination
- Return cursors, not page numbers.
- Cursor is opaque base64-encoded `LastEvaluatedKey`.
- Never accumulate all pages in memory.

## Multi-item writes
- `BatchWriteCommand` for non-transactional bulk inserts (with retry on UnprocessedItems).
- `TransactWriteCommand` for atomic multi-item updates (≤ 100 items, same region).

## Conditional writes
- Always include a `ConditionExpression` on UpdateItem when you require a precondition (item exists, last-writer-wins, version increment).
- Catch `ConditionalCheckFailedException` and translate to a typed domain error.

## GSI design
- Choose key shape to support the query — avoid GSI scans.
- Sparse GSIs only when the index attribute may be absent.
- Document each GSI's purpose in a comment in the Terraform module.

## Marshalling
- `removeUndefinedValues: true` on the Document client (removes accidental undefined → empty-string traps).
- Date values serialised as ISO strings; deserialised at the adapter boundary.
- `Set` types only when set semantics are required; otherwise array of unique strings.
