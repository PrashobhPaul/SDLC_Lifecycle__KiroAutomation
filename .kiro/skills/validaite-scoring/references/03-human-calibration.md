# Human-SME Calibration Job

## When
Sprint 2 (mid-program) and Sprint 5 (pre-handover) per roadmap commitment.

## Inputs
- 20 generated artifacts per sprint, sampled across all 5 agents and all 5 CALM layers.
- Named SMEs from SWA (per Dependencies sheet).
- Same prompt template used by the LLM-judge.

## Procedure
1. AI-COE Lead runs `tools/calibration-prepare.ts --sprint S<n>` which sources 20 artifacts and
   produces a packet at `.kiro/validaite/calibration/S<n>-packet/`.
2. SMEs score each artifact on the same six axes via a simple form.
3. SME scores returned as `.kiro/validaite/calibration/S<n>-sme-scores.json`.
4. `tools/calibration-compute.ts --sprint S<n>` produces:
   - `.kiro/validaite/calibration/S<n>-correlations.json`
   - `.kiro/validaite/calibration/S<n>-report.md`

## Output report contents
- Per-axis Pearson correlation
- Per-axis MAE
- Per-axis bias (judge_mean − sme_mean)
- Composite correlation
- Recommended action: keep / re-prompt / re-train weights

## If correlation < 0.7
- Inspect per-axis: which axis drives the disagreement?
- Update the LLM-judge prompt template under version control (bump version number).
- Re-score the same 20 artifacts with the new prompt.
- If still < 0.7: route to AI-COE Lead for a methodology decision (weight changes, new axis, sample size).

## Sample sizing
- 20 is the minimum for Phase 1; below this the correlation is too noisy.
- Sample stratified: at least 3 per agent, at least 2 per CALM layer.
