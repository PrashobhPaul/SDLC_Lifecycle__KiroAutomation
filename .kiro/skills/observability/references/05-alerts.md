# Alerts & Escalation

## Alert table

| Alert | Threshold | Severity | Owner | Runbook |
|-------|-----------|----------|-------|---------|
| HITL gate skipped | gate_compliance < 100% in last sprint | High | AI-COE Lead | runbooks/hitl-gap.md |
| Token budget breach | per-agent cumulative > budget | Med | Squad | runbooks/token-budget.md |
| DLQ non-empty | depth > 0 for >5 min | High | SRE on-call | runbooks/dlq.md |
| Fallback usage spike | >5x baseline in 1h | Med | Squad | runbooks/fallback.md |
| tsc errors gate | tsc errors > 0 on Developer handoff | High | Developer + Code Reviewer | runbooks/tsc-fail.md |
| Coverage drop | coverage delta < -5% | Med | Test Engineer + Tech Lead | runbooks/coverage.md |
| GraphQL breaking change un-overridden | breaking_changes > 0 without G6 | High | Code Reviewer | runbooks/breaking-change.md |
| Allow-list violation | tool call rejected by data-scope-guard | High | AI-COE Lead + Cyber | runbooks/scope-violation.md |
| Injection block | guardrail injection_blocks > 0 | Med (track) | Guardrail owner | runbooks/injection.md |
| Reviewer FP rate spike | rate > 15% over a sprint | Med | Code Reviewer + Tech Lead | runbooks/fp-calibration.md |
| Hallucination spike | rate > 2% over 24h | High | Squad + AI-COE | runbooks/hallucination.md |
| Human-AI correlation drop | correlation < 0.7 on calibration set | High | AI-COE Lead | runbooks/calibration.md |

## Routing
- High → PagerDuty / Slack #ai-coe-oncall + email AI-COE Lead.
- Medium → Slack #ai-coe-monitoring + ticket created in Jira.
- Low / track-only → dashboard only.

## Suppression windows
- Sprint demo windows: medium alerts batched and reviewed post-demo.
- Maintenance windows declared in `.kiro/validaite/maintenance.yml`.
