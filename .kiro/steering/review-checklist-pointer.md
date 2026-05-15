---
id: review-checklist-pointer
title: Code Review Checklist Pointer
version: 1
applies_to:
  - agent: code-reviewer
    when: always
  - agent: developer
    when: pre-handoff
points_to:
  - kind: knowledge
    path: knowledge/review-checklists/ui-audit.md
    label: UI React/TS audit (sections A–W, 28 rules, tools T1–T20)
  - kind: knowledge
    path: knowledge/review-checklists/be-lambda-audit.md
    label: BE Lambda + Terraform audit (sections A–M, 70 rules, tools T1–T31)
  - kind: skill
    path: skills/frontend-coding-standards/references/17-code-review-checklist.md
    label: Frontend review-rule index with severities
  - kind: skill
    path: skills/backend-coding-standards/references/18-code-review-checklist.md
    label: Backend review-rule index with severities
token_budget:
  load_on_activation_tokens: 800
  cumulative_session_tokens: 12000
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: false
  on_critical_finding: open_g6_or_g9
checksum: pending-linter-recompute
---
# Review Checklist Pointer

Use this pointer to reach the canonical UI and BE-Lambda audit rules without loading
bulk content per turn. The Code Reviewer agent always activates this; the Developer
activates it during the pre-handoff self-check phase.

Severities map to gates: any CRITICAL finding blocks G9 (pre-merge) and triggers G6
(reviewer-blocker-override) review. MAJOR findings count toward sprint AIQ but do not
block merge by themselves; they roll up into the per-agent scorecard.
