---
id: observability
title: Observability Pointer
version: 1
applies_to:
  - agent: feature-builder
    when: always
  - agent: developer
    when: always
  - agent: test-engineer
    when: always
  - agent: code-reviewer
    when: always
  - agent: documentation
    when: always
points_to:
  - kind: skill
    path: skills/observability/SKILL.md
    label: Observability skill
  - kind: schema
    path: schemas/aiq-scorecard.schema.json
    label: AIQ scorecard schema
  - kind: skill
    path: skills/validaite-scoring/SKILL.md
    label: ValidAIte scoring skill
token_budget:
  load_on_activation_tokens: 700
  cumulative_session_tokens: 3500
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: false
checksum: pending-linter-recompute
---
# Observability

Every agent turn emits one AIQ JSONL record. Trace context propagates via the handoff
envelope. Metrics exposed via Prometheus + CloudWatch EMF. Sprint scorecard aggregated
by tools/validaite-aggregate.ts (with --explain for SWA self-service).
