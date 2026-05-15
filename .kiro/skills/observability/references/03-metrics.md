# Metrics Inventory

## Agent-level metrics (emitted by every agent turn)

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| kiro_agent_turn_total | counter | agent, phase, outcome | Total agent turns |
| kiro_agent_turn_duration_ms | histogram | agent, phase | Wall-clock duration |
| kiro_agent_tokens_in | counter | agent, model | Input tokens consumed |
| kiro_agent_tokens_out | counter | agent, model | Output tokens generated |
| kiro_hitl_gate_total | counter | gate, outcome | Gate fires by outcome |
| kiro_guardrail_block_total | counter | rule, severity | Guardrail blocks by rule |
| kiro_handoff_validate_total | counter | from_agent, to_agent, result | Handoff schema validations |

## Verification metrics (emitted from build/test/security tools)

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| kiro_tsc_errors | gauge | feature_id | tsc --noEmit src errors |
| kiro_eslint_errors | gauge | feature_id | ESLint errors |
| kiro_test_pass_rate | gauge | feature_id, layer | Tests passed / total |
| kiro_coverage_pct | gauge | feature_id, layer | Coverage % |
| kiro_tfsec_findings | gauge | severity | tfsec findings |
| kiro_graphql_breaking_changes | gauge | feature_id | breaking-change count |

## Async-orchestrator metrics

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| kiro_queue_depth | gauge | queue | Items waiting |
| kiro_queue_inflight | gauge | queue | Items in-flight |
| kiro_job_attempts | counter | queue, outcome | Attempts |
| kiro_dlq_depth | gauge | queue | DLQ depth (alert if >0) |
| kiro_fallback_used | counter | queue | Times the fallback path was taken |

## Exposition
- Prometheus endpoint at `/metrics` from a sidecar `validaite-exporter`.
- CloudWatch EMF emitted by Lambda-based agent runs.

## Storage
- Short-term (7 days): Prometheus / CloudWatch metric streams.
- Long-term (per sprint): aggregated to `.kiro/validaite/scorecards/<sprint>.json`.
