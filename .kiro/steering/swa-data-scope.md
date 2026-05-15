---
id: swa-data-scope
title: SWA Data Scope Allow-List Pointer
version: 1
applies_to:
  - agent: feature-builder
    when: on-tool-call
    keywords: [confluence, jira, figma, gitlab]
  - agent: developer
    when: on-tool-call
    keywords: [confluence, jira, figma, gitlab]
  - agent: documentation
    when: on-tool-call
    keywords: [confluence, jira, figma, gitlab]
points_to:
  - kind: knowledge
    path: knowledge/swa-stack/allow-list.yml
    label: Cyber-approved Confluence/Jira/Figma/GitLab scopes
  - kind: skill
    path: skills/governance-ethics/references/02-data-scope.md
    label: Data-scope enforcement rules
token_budget:
  load_on_activation_tokens: 800
  cumulative_session_tokens: 3000
governance:
  pii_class: low
  data_scope: [confluence, jira, figma, gitlab]
  hitl_required: true
checksum: pending-linter-recompute
---
# SWA Data Scope

Bounded list of Confluence spaces, Jira projects, Figma workspaces, and GitLab repos
that agents can read. Out-of-scope reads are refused at the agent boundary. Write-back
is HITL-gated. Every tool call against this scope is logged under .kiro/validaite/.
