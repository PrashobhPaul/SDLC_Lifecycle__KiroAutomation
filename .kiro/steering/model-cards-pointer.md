---
id: model-cards-pointer
title: Per-Agent Model Cards Pointer
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
    path: skills/governance-ethics/references/03-model-cards.md
    label: Model assignment table with rationale and review cadence
  - kind: knowledge
    path: knowledge/calm/model-changelog.md
    label: Model change log with calibration impact
token_budget:
  load_on_activation_tokens: 500
  cumulative_session_tokens: 2000
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: false
checksum: pending-linter-recompute
---
# Model Cards Pointer

Records the model assigned to each of the five agents (Feature Builder, Developer,
Test Engineer, Code Reviewer, Documentation), the rationale, and the review cadence.
Any model change requires re-running the human calibration job before scorecards
from the new model are accepted into the sprint roll-up.
