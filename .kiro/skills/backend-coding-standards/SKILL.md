---
name: backend-coding-standards
description: >
  Backend (Node.js TypeScript Lambda) coding standards skill for the SWA CALM platform. Activate
  on any task that creates, modifies, or reviews .ts files under src/ targeting AWS Lambda. Encodes
  hexagonal architecture, AWS SDK v3 usage, DynamoDB / Aurora data access, error handling, structured
  logging, OTEL tracing, idempotency, batchItemFailures, IaC alignment, and the full BE-Lambda audit
  checklist (sections A–M, 70 enforcement rules) used by the Code Reviewer agent.
---

# Backend Coding Standards — Skill

## How to use this skill

| Concern | Reference |
|---------|-----------|
| Existing-project checklist (match patterns; do not impose new-project standards) | `references/00-existing-project-checklist.md` |
| Architecture (hexagonal: domain · application · adapters · ports) | `references/01-architecture.md` |
| TypeScript config and language rules | `references/02-typescript.md` |
| Lambda handler shape | `references/03-lambda-handler.md` |
| AWS SDK v3 usage and DI container | `references/04-aws-sdk-v3.md` |
| DynamoDB access patterns | `references/05-dynamodb-access.md` |
| Aurora / RDS access (Calm context) | `references/06-aurora-access.md` |
| Security (secrets, IAM, TLS, sanitisation) | `references/07-security.md` |
| Error handling and typed errors | `references/08-error-handling.md` |
| Structured logging with Powertools | `references/09-logging.md` |
| OTEL tracing | `references/10-tracing.md` |
| Performance (no `await` in loops, pagination, no full Scan, projection expressions) | `references/11-performance.md` |
| Input validation with Zod | `references/12-input-validation.md` |
| Testing standards | `references/13-testing.md` |
| Dependency hygiene | `references/14-deps.md` |
| Data processing (mappers pure, date parsing, float arithmetic) | `references/15-data-processing.md` |
| Reusability / SRP / no within-lambda duplication | `references/16-reusability.md` |
| Per-file checklist (file-type → BE-audit sections) | `references/17-per-file-checklist.md` |
| Code review checklist (BE-Lambda audit A–M, 70 rules) | `references/18-code-review-checklist.md` |

## Always-on principles

- Domain layer NEVER imports `@aws-sdk`, `axios`, ORMs, or HTTP frameworks (BE rule 10 — CRITICAL).
- `any` is MAJOR per instance (BE rule 11) — use `unknown` + narrowing.
- AWS SDK v3 modular imports only (BE rule 14) — no SDK v2.
- `console.log` outside the structured logger is MAJOR per file (BE rule 15).
- Every async Lambda trigger has a DLQ (BE rule 16).
- X-Ray / OTEL tracing on (BE rule 17).
- SQS consumers return `batchItemFailures` (BE rule 18).
- SQS / SNS / EventBridge mutations carry idempotency keys (BE rule 25).
- Hardcoded secrets / credentials → CRITICAL (BE rule 12).
- `Action: "*"` AND `Resource: "*"` together → CRITICAL (BE rule 13).
- IAM action ↔ SDK call cross-reference (BE rule 61).
