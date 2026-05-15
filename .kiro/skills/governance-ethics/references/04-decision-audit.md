# Decision Audit Log

## Why
Every agent decision that flows into a handoff must be auditable: what was decided, why, by whom (agent + human approver), and against what evidence.

## Format
Each significant decision writes one JSONL line to `.kiro/validaite/telemetry/decisions.jsonl`:

```json
{
  "ts": "2026-05-07T10:24:31.221Z",
  "decision_id": "d_4f9a2c8e",
  "feature_id": "crew-rest-violation-alert",
  "agent": "developer",
  "agent_run_id": "ar_...",
  "trace_id": "tr_...",
  "category": "scope-boundary | architecture | schema-change | iac-change | test-strategy | review-blocker | doc-stale | self-correction",
  "summary": "Added new GraphQL field crewRestStatus to Pilot type; non-breaking optional",
  "evidence_paths": [".kiro/handoff/<id>/plan.md", ".kiro/handoff/<id>/schema-change.md"],
  "hitl_gate": "G3",
  "outcome": "approved",
  "approver": "Marissa@SWA",
  "rationale": "Non-breaking, additive only; required for FAA Part 117 alert",
  "expires_at": null
}
```

## Rules
1. One decision = one line. No batching.
2. `decision_id` is stable; refer to it from artifacts and from sprint reports.
3. Every CRITICAL finding override produces a decision record with `category=review-blocker` and a non-null `expires_at`.
4. Sprint scorecard summarises decisions by category and outcome.
