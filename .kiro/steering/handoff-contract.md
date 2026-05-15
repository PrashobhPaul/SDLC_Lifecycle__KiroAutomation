---
id: handoff-contract
title: Handoff Contract Pointer
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
  - kind: schema
    path: handoff/schemas/handoff-envelope.schema.json
    label: Handoff envelope JSON Schema v1
  - kind: schema
    path: schemas/question-back.schema.json
    label: Question-Back Model schema
  - kind: schema
    path: schemas/hitl-gates.json
    label: 9-gate HITL inventory
  - kind: template
    path: templates/handoff.md
    label: Human-readable handoff mirror
token_budget:
  load_on_activation_tokens: 1000
  cumulative_session_tokens: 4000
governance:
  pii_class: low
  data_scope: [local]
  hitl_required: true
checksum: pending-linter-recompute
---
# Handoff Contract

Every agent-to-agent transition validates against handoff-envelope.schema.json.
Downstream agents refuse to start when validation fails. Question-Back Model formalises
clarification cycles; the literal token "answered" gates resume. The 9 HITL gates are
the governance backbone.
