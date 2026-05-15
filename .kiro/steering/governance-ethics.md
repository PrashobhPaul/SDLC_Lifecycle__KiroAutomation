---
id: governance-ethics
title: Governance & Ethics Pointer
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
    path: skills/governance-ethics/SKILL.md
    label: Governance & Ethics skill
  - kind: schema
    path: schemas/hitl-gates.json
    label: 9-gate HITL inventory
  - kind: knowledge
    path: knowledge/security/red-lines.md
    label: Red-line refusal catalogue
token_budget:
  load_on_activation_tokens: 700
  cumulative_session_tokens: 3500
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: true
checksum: pending-linter-recompute
---
# Governance & Ethics

Encodes the 9 HITL gates, model-card adherence, decision audit, RACI, red lines, bias-fairness
checks for FAA-117 / scheduling features, and the governance scorecard inputs. Every agent
activates this on entry — it is the policy layer.
