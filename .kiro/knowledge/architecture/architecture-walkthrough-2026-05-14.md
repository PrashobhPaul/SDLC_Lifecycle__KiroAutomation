# CALM Architecture Walkthrough — 2026-05-14

SWA Crew-Ops architect (Prashanth) walked through the CALM backend topology.
Decisions and open follow-ups captured here are the
basis for several V3 KB and steering updates.

## Decisions ratified in this meeting

1. **Two AWS accounts back CALM.**
   - Crew Core (DynamoDB) — owns Crew Member profile + sensitive PII.
   - Calm/DHP (Aurora PostgreSQL) — owns Calm-domain data (FMLA, leave events).
2. **AppSync GraphQL is the sole UI→backend gateway.** Each query/mutation
   resolves to a VTL template or a Lambda. No direct DB access from the MFE.
3. **Promotion path is DEV-1 → Golden DEV → QA → Prod.** Agents may target
   DEV-1 only; promotion past DEV-1 is a human action.
4. **Sensitive PII never transits Kafka.** Topics carry identifiers; consumers
   that need PII fetch via the Crew-Member GraphQL API.
5. **MFEs are S3 bundles composed via single-spa.** Lambda authorizer pattern
   handles AD-group → role mapping per AppSync API.
6. **Multi-region is Prod-only:** Route53 weighted routing + Aurora Global DB +
   DynamoDB Global Tables. Failover is operator-triggered, not auto-promoted.
7. **Lambda keep-warm is enabled in all environments**, including DEV-1, so
   agent-built tests do not see cold-start skew.
8. **CSS DMS → Kinesis → EventBridge** CDC pipeline is in progress; consumers
   for `css.staff_bank.cdc.v1` should not yet be agent-generated.

## Audit gaps and risks acknowledged

- Some AppSync data-source mappings live in the AWS console (not in Terraform).
- Lambda authorizer repository URL not yet shared.
- Ops Suite UI integration diagram (single-spa root config + MFE registry)
  not yet shared.

## Confirmed GitLab paths

- Crew-Member API: `https://southwest.gitlab-dedicated.com/csr/contexts/crew-member/application/cm-deploy-module`
- CALM deploy modules: `https://southwest.gitlab-dedicated.com/csr/contexts/calm/app/deploy-modules`
- CALM Confluence: `https://southwest.atlassian.net/wiki/spaces/CPLN/pages/495943879/Crew+Architecture+Crew+Attendance+and+Leave+Management+CALM`

## Impact on V3

- `knowledge/swa-stack/aws-stack.md` — augmented with two-account split, multi-region detail.
- `knowledge/swa-stack/data-classification.md` — NEW; codifies Kafka exclusion.
- `knowledge/swa-stack/deployment-flow.md` — NEW; codifies agent-target constraint.
- `steering/swa-data-scope.md` — updated to reference the new pointer files.
- `steering/calm-architecture-pointer.md` — updated `points_to` to include new files.
- Agent prompts — Feature Builder gains an explicit "data classification" question;
  Code Reviewer gains a "Kafka schema scan" check; Developer is reminded to target DEV-1 only.

## See also

- `knowledge/architecture/calm-architecture.md` — system overview.
- `knowledge/swa-stack/aws-stack.md` — stack reference.
