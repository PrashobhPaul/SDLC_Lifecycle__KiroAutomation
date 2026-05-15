---
id: scope-boundaries
title: Scope Boundaries Pointer
version: 1
applies_to:
  - agent: feature-builder
    when: always
  - agent: developer
    when: on-handoff-receive
  - agent: code-reviewer
    when: always
points_to:
  - kind: knowledge
    path: knowledge/calm/scope-boundaries.md
    label: CALM scope boundaries (in-scope vs out-of-scope; layer ownership)
token_budget:
  load_on_activation_tokens: 600
  cumulative_session_tokens: 2400
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: false
checksum: pending-linter-recompute
---
# Scope Boundaries

The Feature Builder enforces these against every story. Out-of-scope items are flagged
inline; zero silent absorption. Code Reviewer flags PRs that drift across documented
boundaries (e.g., Calm context reaching directly into Crew Member sensitive tables).
