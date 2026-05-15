# Governance Scorecard Inputs

## What feeds the sprint governance scorecard

Per sprint, the ValidAIte aggregator (`tools/validaite-aggregate.ts`) computes:

| Metric | Source | Target |
|--------|--------|--------|
| HITL gate compliance | Count fired G1–G9 / required | 100% |
| Decision audit completeness | Decisions logged / decisions made | 100% |
| Allow-list adherence | Tool calls in scope / total tool calls | 100% |
| Override rate | G6 overrides / blocker findings | ≤ 10% |
| Override expiry adherence | Overrides re-blocked after expiry / total expired | 100% |
| Red-line breach count | Red-line refusals fired by guardrails | track only |
| Bias-fairness flags | Decisions involving protected attribute, with policy ref | track only |
| Model-card adherence | Runs on declared model / total runs | 100% |

## Output
Aggregated to `.kiro/validaite/scorecards/governance-S<n>.json` with a markdown summary at
`.kiro/validaite/scorecards/governance-S<n>.md`.
