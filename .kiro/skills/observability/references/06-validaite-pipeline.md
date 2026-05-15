# ValidAIte Aggregation Pipeline

## Inputs
- `.kiro/validaite/telemetry/<agent>.jsonl` — per-agent JSONL records.
- `.kiro/validaite/telemetry/decisions.jsonl` — decision audit lines.
- `.kiro/validaite/golden-runs/<sprint>/*.json` — golden-set comparisons.
- `.kiro/validaite/calibration/<sprint>.json` — human SME scores for the 20-output calibration set.

## Pipeline (one command, runnable by Prashant unaided — Phase 1 success criterion)

```bash
node tools/validaite-aggregate.ts \
  --sprint S<n> \
  --golden-set .kiro/validaite/golden-runs/S<n>/ \
  --calibration .kiro/validaite/calibration/S<n>.json \
  --out .kiro/validaite/scorecards/S<n>/
```

The aggregator produces per agent and per sprint:
- `<agent>-summary.json` — averaged metrics.
- `<agent>-summary.md` — human-readable summary.
- `governance-S<n>.json` — gates, decisions, allow-list adherence, override expiry.
- `chain-fidelity-S<n>.json` — handshake fidelity per pair (FB→Dev, Dev→Test, etc.).
- `dashboard-S<n>.json` — feeds the Grafana / CloudWatch dashboard.

## Calibration step
For Sprint 2 and Sprint 5 (per roadmap):
- 20 outputs scored by humans against the LLM-as-judge prompt.
- Aggregator computes Pearson correlation and the per-axis delta.
- If correlation < 0.7, calibration job updates the LLM-judge prompt template at `tools/llm-judge-prompt.md`
  and re-runs the scoring job. Report includes both before/after correlations.

## Self-service
The `validaite-aggregate.ts` script accepts `--explain` which prints what each metric means and where it came from.
That's the "Prashant runs this unaided" path.
