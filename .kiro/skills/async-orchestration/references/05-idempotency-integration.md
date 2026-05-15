# Idempotency Integration

## Composition with the guardrail key

The orchestrator does not generate idempotency keys. It receives them from the calling agent, which
generated them via the formula in `guardrails-io-sanity/references/06-idempotency.md`:

```
idempotency_key = sha256(<feature_id> || <agent> || <phase> || <artifact_hash>)
```

This means two retries of the same logical action — even across agent restarts — produce the same key
and are short-circuited correctly.

## Local idempotency table

```
.kiro/validaite/idempotency/<key>.json
{
  "key": "<sha256>",
  "first_seen_at": "<RFC3339>",
  "last_seen_at": "<RFC3339>",
  "outcome": "success | dlq | pending",
  "result": { /* opaque action-specific blob */ },
  "ttl_at": "<RFC3339>"  // 30 days from last_seen_at
}
```

## Behaviour

```ts
async function executeJob(job: Job): Promise<JobResult> {
  const cached = await loadIdempotency(job.idempotency_key);
  if (cached?.outcome === 'success') {
    return { ...cached.result, replayed: true };
  }
  if (cached?.outcome === 'pending') {
    throw new Error('IdempotencyKeyConflict: pending');
  }

  try {
    await markPending(job.idempotency_key);
    const result = await callAction(job);
    await markSuccess(job.idempotency_key, result);
    return { ...result, replayed: false };
  } catch (err) {
    await clearPending(job.idempotency_key);
    throw err;
  }
}
```

## Cross-system support
- Jira: native idempotency key support via `X-Atlassian-Token`-equivalent custom header where supported.
- GitLab: supports idempotency on MR creation via `idempotency_key` header (recent versions). Where unsupported, local table prevails.
- AppSync mutations: receive the key as a request-context attribute and the resolver checks DynamoDB before write.
