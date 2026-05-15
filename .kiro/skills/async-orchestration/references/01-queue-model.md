# Durable Queue Model

## Queues
- `q.handoff`     — agent-to-agent handoffs that can be re-driven
- `q.tool.jira`   — Jira mutations (write-back)
- `q.tool.figma`  — Figma comments
- `q.tool.gitlab` — GitLab MR creation
- `q.notify`     — non-critical notifications (Slack, email)

Phase 1 implementation: file-backed durable queue under `.kiro/validaite/queues/<queue>/`.
Phase 2 (post-pilot, not in scope for June 30): switch to SQS with the same envelope.

## Job envelope

```json
{
  "job_id": "j_<random8>_<unix-ms>",
  "queue": "q.tool.jira",
  "kind": "jira-create-issue | jira-comment | gitlab-mr-create | handoff-redrive | ...",
  "idempotency_key": "<sha256>",
  "payload": { /* action-specific */ },
  "context": {
    "feature_id": "<id>",
    "agent": "<agent-name>",
    "trace_id": "<id>",
    "span_id": "<id>"
  },
  "attempts": 0,
  "max_attempts": 3,
  "backoff": "exponential",
  "next_eligible_at": "<RFC3339>",
  "fallback_path": "<local-artifact-path>",
  "dlq_path": ".kiro/validaite/telemetry/dlq.jsonl",
  "created_at": "<RFC3339>"
}
```

## State transitions
`enqueued → in_flight → (success | failure_retryable | failure_terminal) → (done | dlq)`

## Visibility
Queue depth and in-flight count are exposed in metrics (`kiro_queue_depth`, `kiro_queue_inflight`).
DLQ depth alerts at >0 (operations dashboard).

## Concurrency
Per-queue concurrency cap declared in `.kiro/validaite/queues/config.yml`. Defaults: 4 in-flight per queue.
