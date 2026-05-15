# Per-File Checklist (file-type → BE-audit sections)

Code Reviewer applies this matrix on every BE file:

| File type | Apply sections |
|-----------|---------------|
| Lambda handler (`adapters/inbound/*.ts`) | A · B · C · D · E · F · H · I · J |
| Use case / application service | A · B · C · F · G · H · J · K · M |
| Domain entity / value object | A · B · C · F · K · M |
| Port (interface) | A · B · C · M |
| Adapter (AWS, HTTP, DB) | A · B · C · E · F · G · H · J · K · M |
| Terraform `.tf` | E · I |
| Test file | L |
| Config (`package.json`, `tsconfig.json`, ESLint config) | J |

Section legend (BE-audit):
- **A** Code quality
- **B** TypeScript
- **C** Hexagonal architecture
- **D** Lambda handler
- **E** Security
- **F** Error handling & observability
- **G** Performance & scalability
- **H** Input validation
- **I** Infrastructure (Terraform)
- **J** Dependency & config hygiene
- **K** Data processing
- **L** Testing
- **M** Reusability, maintainability, scalability

Cross-file checks (Step 3 of the audit) — DRY within same lambda only, layer violations, circular deps,
dead exports — are not per-file; they roll up into `REVIEW-arch.md`.

## Severity matrix per section

| Section | Typical severity | Common findings |
|---------|------------------|-----------------|
| A | MAJOR | files >200 LOC, functions >25 LOC, magic strings, dead code |
| B | MAJOR | `any`, missing return types, unsafe casts, `!` non-null |
| C | CRITICAL or MAJOR | domain importing infra (CRITICAL); local interface duplicating domain (MAJOR) |
| D | MAJOR | event typed as `any`, validation duplicated, SDK in handler body |
| E | CRITICAL | hardcoded secret, IAM `*` w/ `*`, missing KMS, open SG |
| F | MAJOR | empty catch, missing X-Ray, no DLQ, error masking |
| G | MAJOR | `await` in loop, full Scan, N+1, `Promise.all` where allSettled needed |
| H | MAJOR (CRITICAL when SQL injection) | dynamic query, missing schema validation |
| I | CRITICAL or MAJOR | hardcoded region/account, IAM wildcard, deprecated runtime |
| J | MAJOR | unused deps, missing pre-commit, committed `.env`, Renovate absent |
| K | MAJOR | mapper side effects, date-only parsed via `new Date()` |
| L | MAJOR | coverage <80%, snapshot tests on logic, mock type drift |
| M | MAJOR | within-lambda DRY, hardcoded business constants, dead factory |
