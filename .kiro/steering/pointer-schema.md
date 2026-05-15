---
id: pointer-schema
title: Pointer Schema Pointer
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
    path: pointer-schema/POINTER_SCHEMA.md
    label: Pointer Schema v1 (locked)
token_budget:
  load_on_activation_tokens: 400
  cumulative_session_tokens: 1500
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: false
checksum: pending-linter-recompute
---
# Pointer Schema

Steering files are pointer-only YAML frontmatter + short rationale. Bulk content lives in
`.kiro/knowledge/` or `.kiro/skills/<skill>/references/`. The linter validates frontmatter,
checksum, and token budgets. Schema v1 locked 2026-04-26; breaking changes bump major.
