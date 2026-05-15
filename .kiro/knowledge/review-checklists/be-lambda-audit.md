# Backend Lambda + Terraform Audit — Canonical Checklist

Canonical BE review checklist used by the Code Reviewer agent for any change
touching `node-resolver` or `terraform-iac`. Sections A–M with 70 enforcement
rules. Tools T1–T31 are run in parallel by the Code Reviewer's tooling pass.

## Severity

- **CRITICAL** — blocks G9 and auto-triggers G6.
- **MAJOR** — rolls into per-agent AIQ.
- **MINOR** — informational.

## Tools (parallel)

| Tool      | Purpose                                                        |
|-----------|----------------------------------------------------------------|
| T1        | tsc --noEmit                                                   |
| T2        | eslint --max-warnings=0                                        |
| T3        | vitest --coverage                                              |
| T4        | npm audit                                                      |
| T5        | semgrep                                                        |
| T6        | gitleaks                                                       |
| T7        | depcheck                                                       |
| T8        | ts-prune                                                       |
| T9        | madge --circular                                               |
| T10       | size-limit (Lambda zip)                                        |
| T11       | aws-sdk-client-mock test pass                                  |
| T12       | tfsec                                                          |
| T13       | tflint                                                         |
| T14       | checkov                                                        |
| T15       | terraform validate / fmt                                       |
| T16       | terraform plan diff (no destroy without G4)                    |
| T17       | iam-sdk-crossref.ts (rule 61)                                  |
| T18–T22   | Powertools log/trace/metrics presence checks                   |
| T23       | DLQ presence on async triggers                                 |
| T24       | Idempotency-key check on SQS/SNS/EB consumers                  |
| T25       | RDS Proxy required check                                       |
| T26       | Secrets Manager rotation enabled check                         |
| T27       | KMS multi-region key check                                     |
| T28       | Aurora Global presence check                                   |
| T29       | DynamoDB Global Tables presence check                          |
| T30       | Region/account/ARN hardcoding scan                             |
| T31       | Domain-layer `@aws-sdk` import scan (rule 10 CRITICAL)         |

## Rule Index (selected — full set in skill ref 18)

| #  | Section | Rule                                                                             | Severity  |
|----|---------|----------------------------------------------------------------------------------|-----------|
| 10 | C       | Domain-layer files do not import `@aws-sdk/*`                                    | CRITICAL  |
| 11 | C       | No `any` types introduced                                                        | MAJOR     |
| 13 | E       | No region / account / ARN hardcoded in Terraform                                 | CRITICAL  |
| 14 | C       | AWS SDK v3 only (no v2 imports)                                                  | MAJOR     |
| 15 | F       | No `console.log` (Powertools logger required)                                    | MAJOR     |
| 16 | E       | DLQ configured on every async-trigger Lambda                                     | CRITICAL  |
| 17 | F       | X-Ray / OTEL tracing enabled (Powertools Tracer)                                 | MAJOR     |
| 18 | C       | SQS handler returns `batchItemFailures`                                          | CRITICAL  |
| 21 | H       | No unbounded `Promise.all` over external calls                                   | MAJOR     |
| 25 | C       | Idempotency-key applied on SQS/SNS/EventBridge consumers                         | CRITICAL  |
| 27 | D       | DynamoDB queries use ProjectionExpression                                        | MAJOR     |
| 29 | C       | AWS clients are singleton with lazy-init outside the handler                     | MAJOR     |
| 33 | D       | DynamoDB pagination uses cursor (LastEvaluatedKey)                               | MAJOR     |
| 39 | D       | RDS Proxy used for all Aurora connections from Lambda                            | CRITICAL  |
| 44 | A       | Hexagonal layering: domain / application / ports / adapters                      | MAJOR     |
| 47 | C       | All Zod schemas applied at the adapter boundary                                  | MAJOR     |
| 49 | F       | Validation runs once per request — handler **or** use case, not both             | MAJOR     |
| 51 | C       | Handler is thin: parse → use case → response                                     | MAJOR     |
| 52 | C       | Errors thrown as typed domain errors; translated at adapter boundary             | MAJOR     |
| 61 | E       | IAM-action ↔ SDK-call cross-reference passes (no over-grant)                     | CRITICAL  |
| 68 | A       | Module boundaries respect dependency direction (no domain → adapter)             | MAJOR     |
| 70 | A       | No within-Lambda duplication of code in > 2 files                                | MAJOR     |

(Full 70-rule table is in `skills/backend-coding-standards/references/18-code-review-checklist.md`.)
