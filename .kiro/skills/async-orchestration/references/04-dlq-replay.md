# DLQ & Replay

## DLQ format
Append-only JSONL at `.kiro/validaite/telemetry/dlq.jsonl`:

```json
{
  "dlq_id": "dlq_<random8>",
  "ts": "<RFC3339>",
  "job_id": "j_...",
  "queue": "q.tool.jira",
  "kind": "jira-create-issue",
  "idempotency_key": "<sha256>",
  "attempts": 3,
  "last_error": { "message": "...", "code": "...", "classification": "non-retryable" },
  "payload": { /* original */ },
  "context": { "feature_id": "...", "agent": "...", "trace_id": "..." }
}
```

## Triage

```bash
node tools/dlq-triage.ts --since 24h
# prints summary by kind, top-N error classes, oldest pending
```

## Replay

```bash
node tools/dlq-replay.ts --kind jira-create-issue --since 6h --confirm
# requires explicit --confirm; prints what would replay first when omitted
# replays preserve idempotency_key — duplicate landings are short-circuited
```

## Replay safety
- Replay is HITL-only (operator action). No agent triggers replay autonomously.
- Replay re-emits the original idempotency key; the local idempotency table prevents double-execution.
- Replay records a decision-audit line with `category=replay`.

## Retention
- DLQ: 30 days, then archived to long-term storage.
- Idempotency table: 30 days, then archived.

## Alerts
- `kiro_dlq_depth > 0` → page operations on-call.
- `kiro_dlq_depth growth rate > N/hour` → escalate to AI-COE Lead.
