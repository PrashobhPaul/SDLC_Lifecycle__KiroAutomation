---
id: guardrails-io-sanity
title: Guardrails & I/O Sanity Pointer
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
    path: skills/guardrails-io-sanity/SKILL.md
    label: Guardrails & I/O Sanity skill
  - kind: schema
    path: handoff/schemas/handoff-envelope.schema.json
    label: Handoff envelope JSON Schema (input/output validation)
  - kind: schema
    path: schemas/aiq-scorecard.schema.json
    label: AIQ scorecard schema (telemetry)
  - kind: schema
    path: schemas/question-back.schema.json
    label: Question-Back Model schema
token_budget:
  load_on_activation_tokens: 800
  cumulative_session_tokens: 4000
governance:
  pii_class: medium
  data_scope: [local]
  hitl_required: true
checksum: pending-linter-recompute
---
# Guardrails & I/O Sanity

Input check at every Phase 0; output check before every artifact write or handoff. Halts on
schema-validation failure, secret detection, or red-line breach. PII redacted at the boundary.
Idempotency keys generated for every external mutation.
