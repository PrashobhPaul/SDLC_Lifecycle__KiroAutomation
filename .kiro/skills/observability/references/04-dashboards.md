# Dashboards

## Sprint Scorecard Dashboard (lead view)
Tiles:
1. HITL gate compliance (target 100%)
2. Compilation pass rate (target ≥98%)
3. Pattern adherence avg (target ≥4.0)
4. Hallucination rate (target ≤2%)
5. Handshake fidelity (target ≥99%)
6. Coverage % on generated code (target ≥80%)
7. Reviewer FP rate (target ≤15%)
8. Token reduction vs v1 (target ≥30%)
9. First-pass acceptance (target ≥60%)
10. Human ↔ AI correlation (target ≥0.7)

## Per-Agent Dashboard
Tiles:
1. Turns per phase (stacked)
2. Token in/out per turn (histogram)
3. Latency p50/p95/p99
4. HITL gates by outcome
5. Guardrail blocks by rule
6. Verification pass rates per tool

## Operations Dashboard (engineer view)
Tiles:
1. Queue depth per queue (alert at threshold)
2. DLQ depth (any non-zero alerts)
3. Async retries per minute
4. Fallback usage per minute
5. Idempotency-skip count
6. Token-budget breaches per agent

## Storage
JSON definitions versioned at `.kiro/validaite/dashboards/`:
- `sprint-scorecard.grafana.json`
- `per-agent.grafana.json`
- `operations.grafana.json`
- CloudWatch equivalents alongside.
