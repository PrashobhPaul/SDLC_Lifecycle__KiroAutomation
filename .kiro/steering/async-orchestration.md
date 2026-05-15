---
id: async-orchestration
title: Async Orchestration Pointer
version: 1
applies_to:
  - agent: feature-builder
    when: on-tool-call
    keywords: [jira-write, confluence-write, gitlab-mr, figma-comment]
  - agent: developer
    when: on-tool-call
    keywords: [jira-write, confluence-write, gitlab-mr, figma-comment, terraform-validate]
  - agent: test-engineer
    when: on-tool-call
    keywords: [jira-write, gitlab-mr]
  - agent: code-reviewer
    when: on-tool-call
    keywords: [jira-write, gitlab-mr]
  - agent: documentation
    when: on-tool-call
    keywords: [confluence-write, jira-write, gitlab-mr]
points_to:
  - kind: skill
    path: skills/async-orchestration/SKILL.md
    label: Async Orchestration skill
  - kind: skill
    path: skills/guardrails-io-sanity/references/06-idempotency.md
    label: Idempotency key formula
token_budget:
  load_on_activation_tokens: 600
  cumulative_session_tokens: 3000
governance:
  pii_class: low
  data_scope: [local, jira, confluence, gitlab, figma]
  hitl_required: true
checksum: pending-linter-recompute
---
# Async Orchestration

Every external mutation goes through a durable queue with 3× exponential backoff (1s/4s/16s),
fallback path on terminal failure, and DLQ routing after max attempts. Replay is operator-only
via tools/dlq-replay.ts. Idempotency keys prevent duplicate landings.
