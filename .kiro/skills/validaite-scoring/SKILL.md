---
name: validaite-scoring
description: >
  ValidAIte scoring skill encoding the AIQ scoring algorithm, the LLM-as-judge calibration prompt,
  the human-SME calibration job, the golden-set comparison procedure, and the sprint scorecard
  aggregation contract for the Kiro CALM construction loop. Activate this skill at the end of every
  agent turn that produces a verifiable artifact (code, schema, IaC, test, doc) and during the
  sprint-end aggregation. Always activated for the five CALM agents.
---

# ValidAIte Scoring — Skill

## How to use this skill

| Concern | Reference |
|---------|-----------|
| AIQ scoring algorithm and weights | `references/01-aiq-algorithm.md` |
| LLM-as-judge prompt template + calibration | `references/02-llm-judge.md` |
| Human-SME calibration job (Sprint 2 + Sprint 5) | `references/03-human-calibration.md` |
| Golden-set comparison procedure | `references/04-golden-set.md` |
| Sprint scorecard aggregation contract | `references/05-scorecard-contract.md` |

## Always-on rules
1. Every verifiable artifact gets an AIQ score on the same 1–5 scale.
2. The LLM-as-judge runs against the same prompt across sprints (versioned).
3. Human SME scores are the ground truth for calibration; correlation target ≥ 0.7.
4. Golden-set runs are deterministic; non-deterministic outputs are flagged and excluded.
5. The aggregator script exposes `--explain` so SWA can self-serve the report.
