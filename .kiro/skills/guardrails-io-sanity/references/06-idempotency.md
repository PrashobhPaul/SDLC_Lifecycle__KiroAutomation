# Idempotency Keys

## When required
Every external mutation must carry an idempotency key:
- Jira ticket creation / update
- Confluence comment write-back
- Figma comment
- GitLab MR creation
- Async-orchestrator job retries
- Self-correction patch apply

## Format
`idempotency_key = sha256(<feature_id> || <agent> || <phase> || <artifact_hash>)`

The same key on a retry is recognised by the receiving system (where supported). Where the receiving
system does not support idempotency natively, the orchestrator maintains a local idempotency table at
`.kiro/validaite/idempotency/<key>.json` recording the result of the first successful call. Subsequent
retries with the same key short-circuit to the recorded result.

## Async retry safety
Combined with the async-orchestration retry policy (3× exponential backoff), idempotency keys ensure
the same logical action is never executed twice. This is the foundation for "Failed jobs recoverable
without duplication" in the Phase 1 success criteria.

## Worked example
Feature Builder writes back a Jira decomposition. On flake:
1. Attempt 1: POST to Jira fails (network).
2. Orchestrator retries with same idempotency key.
3. Attempt 2: succeeds.
4. The local idempotency table records the result.
5. If a third retry fires (orchestrator bug), it short-circuits — no duplicate Jira ticket.

## Storage
- Local table: `.kiro/validaite/idempotency/` (JSON files keyed by `idempotency_key`).
- Retention: 30 days, then archived.
