# Performance & Scalability

## Hard rules (mapped to BE audit)

| Rule | Severity | Source |
|------|----------|--------|
| `await` inside loops | MAJOR per instance | BE rule 21 |
| Sequential `await` chains where parallel is safe | MAJOR | BE checklist G |
| `Promise.allSettled` where partial failure acceptable | MAJOR if missing | BE rule 28 |
| Unbounded in-memory accumulation across pagination | MAJOR | BE rule 26 |
| In-memory pagination (`.slice` on full DB result) | MAJOR | BE rule 32 |
| In-memory filtering on full DB result | MAJOR | BE checklist G |
| DynamoDB full-table `Scan` | MAJOR | BE rule 27 |
| GSI scan fallback (no key condition) | MAJOR | BE checklist G |
| N+1 (query list → one call per item) | MAJOR | BE rule 33 |
| `JSON.parse`/`stringify` on large objects in hot path | MAJOR | BE checklist G |
| Large payload (>1MB) not offloaded to Step Functions / streaming | MAJOR | BE checklist G |
| Response payload exceeds API GW 10MB | MAJOR | BE checklist G |
| Blocking sync operations in async handlers | MAJOR | BE checklist G |

## Right shape, not just less code

```ts
// ❌ N+1
for (const id of ids) {
  results.push(await repo.findById(id));   // BE rule 21 + rule 33
}

// ✅ batched
const out = await ddb.send(new BatchGetCommand({
  RequestItems: {
    [tableName]: { Keys: ids.map((id) => ({ pk: `PILOT#${id}`, sk: 'PROFILE' })) }
  }
}));
```

```ts
// ❌ partial failure swallowed
const results = await Promise.all(events.map((e) => publish(e)));

// ✅ partial failure handled
const settled = await Promise.allSettled(events.map((e) => publish(e)));
const failed = settled.flatMap((r, i) => r.status === 'rejected' ? [{ idx: i, err: r.reason }] : []);
if (failed.length) logger.warn('publish-partial-failure', { failedCount: failed.length });
```

## DynamoDB patterns
- Compose access patterns at design time. Document each in the table's Terraform module README.
- Use `ProjectionExpression` for any read where the consumer needs only a subset of attributes.
- `BatchGetItem` and `BatchWriteItem` capped per AWS limits — chunk and retry on UnprocessedItems.
- Sparse GSIs only when the index attribute may be absent.

## Aurora patterns
- Push sort/filter/aggregate to the database. Lambda memory is not a query engine.
- Use prepared statements; reuse the same client.
- Cursor-based reads for >1000 row results; never load the entire set into memory.

## Lambda
- Provisioned concurrency for latency-critical resolvers (justified case-by-case).
- Reserved concurrency to prevent runaway scale on critical paths.
- Cold-start budget tracked; bundle size target <10MB after tree-shaking.
- Prewarm via the warm-keeper schedule (per meeting notes).
