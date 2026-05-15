# AIQ Scoring Algorithm

## Six axes (1–5 each)

| Axis | What it measures |
|------|------------------|
| pattern_adherence | Match to project patterns and skill references |
| correctness | Compilation + tests pass + verification clean |
| completeness | Coverage of acceptance criteria + edge cases |
| security | Guardrail verdict + tfsec/semgrep clean + IAM least-privilege |
| maintainability | LOC per file/function · DRY · cohesion · readability |
| handshake_fidelity | Downstream agent accepted handoff on first try |

## Weights (Phase 1 default — versioned at `tools/aiq-weights.json`)

```json
{
  "pattern_adherence": 0.20,
  "correctness":       0.25,
  "completeness":      0.15,
  "security":          0.20,
  "maintainability":   0.10,
  "handshake_fidelity":0.10
}
```

## Composite

```
AIQ = round( sum(axis_score * weight), 2 )   in [1.00, 5.00]
```

## Sources per axis

| Axis | Primary source | Secondary |
|------|---------------|-----------|
| pattern_adherence | LLM-judge against skill references | Code Reviewer pattern findings |
| correctness | Verification block on AIQ record | Test results from CI |
| completeness | AC checklist coverage | Edge-case enumeration |
| security | Guardrail verdict + tfsec + semgrep | npm audit |
| maintainability | LOC counters + complexity rules + ts-prune | Code Reviewer findings |
| handshake_fidelity | Did downstream agent start without re-prompting? | Schema validation pass |

## Artifact-class targets

| Artifact | Target AIQ |
|----------|-----------|
| React component | ≥ 4.0 |
| GraphQL schema change | ≥ 4.2 |
| Lambda resolver | ≥ 4.0 |
| VTL template | ≥ 3.8 |
| Terraform module | ≥ 4.2 |
| Test file | ≥ 4.0 |
| Doc file | ≥ 3.5 |

Artifacts below target trigger a Code Reviewer follow-up and feed into the sprint scorecard.
