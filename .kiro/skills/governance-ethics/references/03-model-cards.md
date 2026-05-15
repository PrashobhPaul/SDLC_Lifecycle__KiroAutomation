# Model Cards & Approved Models

## Approved per agent (Phase 1)

| Agent | Model | Rationale |
|-------|-------|-----------|
| feature-builder | claude-sonnet-4-6 | High-quality structured decomposition; cost-effective for repeated runs |
| developer | claude-opus-4-6 | Multi-layer code generation (FE/GQL/BE/VTL/IaC); needs deep reasoning |
| test-engineer | claude-sonnet-4-6 | Test pattern application + coverage analysis |
| code-reviewer | claude-opus-4-6 | Architectural + security review require strongest reasoning |
| documentation | claude-sonnet-4-6 | Pattern-driven generation, schema introspection |

## Card-adherence rules

1. Every agent declares its model in `agent.json`. The runtime refuses to start if the model differs.
2. Model upgrades require a sprint-scope decision; documented in `.kiro/knowledge/calm/model-changelog.md`.
3. AIQ record includes the model used. Calibration jobs are model-aware.

## Off-limits (Phase 1)
- No fine-tuned models without a separate governance review.
- No locally-hosted open-weight models; tool-call surface area not yet validated.
- No model swap mid-sprint without a regression run vs the golden set.
