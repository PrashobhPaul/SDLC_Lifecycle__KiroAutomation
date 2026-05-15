# Review Summary — Code Reviewer Template

## Review <review_id>

**PR:** <pr_url>
**Reviewer agent:** code-reviewer
**Verdict:** approve-with-comments / changes-requested / blocked

## Top-line metrics

| Metric                           | Value | Target | Status |
|----------------------------------|-------|--------|--------|
| TS errors                        |       | 0      |        |
| ESLint warnings                  |       | 0      |        |
| Coverage (overall)               |       | ≥ 80%  |        |
| CRITICAL findings                |       | 0      |        |
| MAJOR findings                   |       | < 5    |        |
| GraphQL breaking changes flagged |       | n/a    |        |
| Terraform destroy in plan        |       | none   |        |

## Findings by severity

- CRITICAL: <count> — listed in `review-files.md`
- MAJOR: <count>
- MINOR: <count>

## Architectural observations

See `review-arch.md` for detailed architectural notes. Key observations:
- <bullet>

## HITL gate recommendation

- G6 (override): <required / not required>
- G9 (pre-merge): <recommend approve / recommend block>
