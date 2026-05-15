# LLM-as-Judge Prompt & Calibration

## Prompt template (versioned at `tools/llm-judge-prompt.md`)

```
You are a senior engineer scoring an AI-generated code artifact for the SWA CALM platform.

Score the artifact on these six axes (1=poor, 5=excellent):
1. Pattern adherence — matches project patterns from <skill-references>
2. Correctness — compiles, type-checks, tests pass, verification clean
3. Completeness — covers all acceptance criteria + reasonable edge cases
4. Security — no secrets, no IAM over-privilege, no injection vectors
5. Maintainability — LOC per file/function, DRY, cohesion, readability
6. Handshake fidelity — downstream agent accepts the handoff without re-prompting

Inputs:
- Artifact: <artifact-path-or-blob>
- Skill references the agent was supposed to follow: <list>
- Acceptance criteria: <AC-list>
- Verification results: <tsc/eslint/coverage/tfsec/semgrep>
- Downstream agent reaction: <accepted | rejected | re-prompt>

Output JSON ONLY (no prose):
{
  "pattern_adherence": <int 1-5>,
  "correctness":       <int 1-5>,
  "completeness":      <int 1-5>,
  "security":          <int 1-5>,
  "maintainability":   <int 1-5>,
  "handshake_fidelity":<int 1-5>,
  "rationale": "<= 200 chars>"
}
```

## Calibration job
Sprint 2 and Sprint 5 (per roadmap): 20 outputs per sprint scored by both LLM-judge and named SME.
Aggregator computes:
- Pearson correlation (per-axis and composite)
- Per-axis MAE (mean absolute error)
- Bias (LLM-judge mean − SME mean)

## Acceptance
- Composite Pearson ≥ 0.7 → accepted, no prompt change.
- Per-axis Pearson ≥ 0.6 → accepted with a watch.
- Below threshold → run a calibration step (see `03-human-calibration.md`).

## Versioning
- Prompt is versioned. Each AIQ record carries the `judge_prompt_version`.
- Sprint-over-sprint comparisons account for prompt version (no apples-to-oranges).
