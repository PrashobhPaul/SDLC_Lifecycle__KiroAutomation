---
id: calm-architecture-pointer
title: CALM Architecture Pointer
version: 2
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
  - kind: knowledge
    path: knowledge/architecture/calm-architecture.md
    label: CALM 5-layer architecture overview
  - kind: knowledge
    path: knowledge/swa-stack/aws-stack.md
    label: SWA AWS stack (AppSync, Lambda, Dynamo, Aurora, multi-region)
  - kind: knowledge
    path: knowledge/swa-stack/bounded-contexts.md
    label: Bounded contexts (Crew Member, Calm, Crew Schedule)
  - kind: knowledge
    path: knowledge/swa-stack/data-classification.md
    label: Data classification tiers + Kafka exclusion rule (2026-05-14)
  - kind: knowledge
    path: knowledge/swa-stack/deployment-flow.md
    label: DEV-1 -> Golden DEV -> QA -> Prod (agents target DEV-1 only)
  - kind: knowledge
    path: knowledge/architecture/architecture-walkthrough-2026-05-14.md
    label: 2026-05-14 walkthrough findings + F1-F3 open items
token_budget:
  load_on_activation_tokens: 1600
  cumulative_session_tokens: 8000
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: false
checksum: pending-linter-recompute
---
# CALM Architecture Pointer

Use this pointer to reach the canonical description of CALM and the SWA AWS stack
without loading bulk content on every turn. Activates always for the five agents.

The architecture has five layers: react-mfe → graphql-schema → node-resolver → vtl-resolver
→ terraform-iac. Bounded contexts are Crew Member, Calm, and Crew Schedule. Multi-region
failover is via Route53 + Aurora Global + Kafka MM2. Lambda authorizer fronts every API.
