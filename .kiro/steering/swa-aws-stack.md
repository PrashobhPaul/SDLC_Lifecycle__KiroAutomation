---
id: swa-aws-stack
title: SWA AWS Stack Pointer
version: 1
applies_to:
  - agent: developer
    when: on-keyword
    keywords: [appsync, lambda, dynamodb, aurora, terraform, kafka, multi-region, vtl, iac]
  - agent: code-reviewer
    when: on-keyword
    keywords: [appsync, lambda, dynamodb, aurora, terraform, kafka, multi-region, vtl, iac]
  - agent: test-engineer
    when: on-keyword
    keywords: [appsync, lambda, dynamodb, aurora, vtl, integration-test, kafka]
points_to:
  - kind: knowledge
    path: knowledge/swa-stack/aws-stack.md
    label: Full SWA AWS architecture (AppSync, Lambda, DDB+Aurora, Kafka, multi-region)
  - kind: knowledge
    path: knowledge/swa-stack/multi-region.md
    label: Multi-region failover details
  - kind: skill
    path: skills/graphql-appsync-standards/SKILL.md
    label: GraphQL+AppSync standards
  - kind: skill
    path: skills/terraform-aws-standards/SKILL.md
    label: Terraform AWS standards
  - kind: skill
    path: skills/event-driven-aws-standards/SKILL.md
    label: Event-driven AWS standards
  - kind: skill
    path: skills/vtl-resolver-standards/SKILL.md
    label: VTL resolver standards
token_budget:
  load_on_activation_tokens: 2000
  cumulative_session_tokens: 12000
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: false
checksum: pending-linter-recompute
---
# SWA AWS Stack Pointer

Activates whenever a turn touches the AWS-specific surface area. Two AWS accounts (Crew Core
with DynamoDB; Calm/DHP with Aurora). AppSync GraphQL APIs front every consumer. Lambda
authorizer enforces AD-group + role checks. Kafka integrates Workday/PDOS/CSS. Multi-region
failover through Route53, Aurora Global, and Kafka MirrorMaker2.
