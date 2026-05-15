# AIQ JSONL Records

## Where
`.kiro/validaite/telemetry/<agent>.jsonl` — append-only, one line per agent turn.

## Schema
Defined in `.kiro/schemas/aiq-scorecard.schema.json`. Every line MUST validate.

## Required fields per turn

```json
{
  "record_version": "1.0.0",
  "ts": "<RFC 3339>",
  "agent": "<agent-name>",
  "agent_run_id": "ar_<random>",
  "trace_id": "tr_<random>",
  "span_id": "sp_<random>",
  "feature_id": "<id>",
  "sprint": "S<n>",
  "phase": "intake | clarify | plan | execute | verify | handoff | review | merge | doc",
  "model": "<model id>",
  "tokens": { "in": 0, "out": 0, "cumulative_session": 0, "budget_remaining": 0 },
  "latency_ms": 0,
  "hitl": { "gate_fired": "G<n>|none", "outcome": "approved|rejected|modified|n/a", "approver": "<name>", "approval_ms": 0 },
  "guardrails": { "input_pass": true, "output_pass": true, "pii_redactions": 0, "injection_blocks": 0, "schema_validation_errors": 0 },
  "verification": { "tsc_no_emit": "pass|fail|n/a", "eslint": "pass|fail|n/a", "graphql_codegen": "pass|fail|n/a", "terraform_validate": "pass|fail|n/a", "tfsec": "pass|fail|n/a", "vtl_simulator": "pass|fail|n/a", "tests_passed": 0, "tests_failed": 0, "coverage_pct": 0 },
  "quality_signals": { "compilation_pass_rate": 0, "pattern_adherence_1_5": 0, "hallucination_rate": 0, "handshake_fidelity": 0, "first_pass_acceptance": false, "modification_density_per_100": 0, "reviewer_false_positive_rate": 0, "human_judge_correlation": 0, "token_reduction_vs_v1_pct": 0 },
  "async": { "queue": "", "attempts": 1, "fallback_used": false, "dlq_routed": false, "idempotency_key": "" },
  "notes": ""
}
```

## Emission rules
1. Emit at every phase transition (intake → clarify → plan → execute → verify → handoff).
2. Emit on every HITL gate fire.
3. Emit on every async retry, fallback, DLQ route.
4. Emit on every guardrail block.
5. Never emit PII. Use `tools/redactor.ts` on free-text `notes`.

## Phase-1 target metrics rolled up from these records

| Metric | Target | Source field |
|--------|--------|--------------|
| HITL gate compliance | 100% | hitl.gate_fired |
| Compilation pass rate | ≥98% | verification.tsc_no_emit |
| Pattern adherence | ≥4.0 | quality_signals.pattern_adherence_1_5 |
| Hallucination rate | ≤2% | quality_signals.hallucination_rate |
| Handshake fidelity | ≥99% | quality_signals.handshake_fidelity |
| Test coverage | ≥80% | verification.coverage_pct |
| Reviewer FP rate | ≤15% | quality_signals.reviewer_false_positive_rate |
| Token reduction vs v1 | ≥30% | quality_signals.token_reduction_vs_v1_pct |
| First-pass acceptance | ≥60% | quality_signals.first_pass_acceptance |
| Human-AI correlation | ≥0.7 | quality_signals.human_judge_correlation |
