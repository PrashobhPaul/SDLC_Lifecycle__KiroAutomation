---
name: terraform-aws-standards
description: >
  Terraform IaC standards skill for the SWA CALM platform. Activate this skill on any task that
  creates, modifies, or reviews .tf or .tfvars files. Covers module structure, remote state with
  S3+DynamoDB lock, multi-region patterns (US East + US West, Aurora Global, Route53 failover),
  IAM least-privilege with action↔SDK cross-reference, Lambda runtime/architecture defaults,
  CloudWatch retention, encryption, tagging, and the tfsec/tflint/checkov gate.
---

# Terraform + AWS Standards — Skill

## How to use this skill

| Concern | Reference |
|---------|-----------|
| Module structure, naming, file split | `references/01-module-structure.md` |
| Remote state (S3 + DynamoDB lock), per-env separation | `references/02-state-management.md` |
| Lambda defaults: runtime, architecture, env vars, layers, bundle | `references/03-lambda-defaults.md` |
| Multi-region failover (Route53, Aurora Global, Kafka MM2) | `references/04-multi-region.md` |
| IAM least-privilege + action↔SDK cross-reference | `references/05-iam-least-privilege.md` |
| Review checklist (tfsec/tflint/checkov) | `references/06-review-checklist.md` |

## Always-on principles

- No hardcoded region, account ID, ARN, or env-specific value (BE rule 13 — CRITICAL).
- IAM `Action: "*"` with `Resource: "*"` is never both at the same time. One side is specific.
- Every CloudWatch log group has a retention.
- Every async Lambda trigger has a DLQ.
- Multi-region is the default for stateful resources; single-region must be justified.
- Tags are mandatory and consistent: `Project`, `Environment`, `Owner`, `BoundedContext`, `CostCenter`.
