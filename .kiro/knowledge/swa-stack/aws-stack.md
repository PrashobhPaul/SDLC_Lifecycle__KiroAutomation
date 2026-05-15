# SWA AWS Stack — Architecture Reference

This is the canonical description of the SWA Crew-Ops AWS stack as scoped for the
CALM Phase-1 pilot. All five agents activate this file via the
`swa-aws-stack` steering pointer.

## High-Level Topology

```
                      ┌─────────────────────────────────┐
                      │        Ops Suite AWS acct       │
   Browser  ──HTTPS──▶│  CloudFront  →  S3 (MFE bundle) │
                      │      │                          │
                      │      ▼                          │
                      │  Single-spa root config         │
                      └────────────┬────────────────────┘
                                   │ GraphQL over HTTPS
                                   ▼
              ┌──────────────────────────────────────────┐
              │  AppSync (Calm/DHP AWS account)          │
              │  Auth: Lambda authorizer                 │
              │       (token + AD groups + roles)        │
              │  Resolvers: VTL  +  Lambda (Node 22 ARM) │
              └────┬──────────────────────┬──────────────┘
                   │                      │
                   │                      │
   ┌───────────────▼────────┐   ┌─────────▼──────────────────┐
   │ Calm/DHP — Aurora PG   │   │ Crew Core acct — DynamoDB  │
   │ (FMLA, Calm domain)    │   │ (Crew Member profile)      │
   │ Multi-region: Global   │   │ Multi-region: Global Tables│
   └────────────────────────┘   └──────────────┬─────────────┘
                                               │
                                               │ via dedicated
                                               │ Crew Member GraphQL API
                                               │ (sensitive info only)
                                               ▼
                          ┌──────────────────────────┐
                          │  Sensitive-info pipeline │
                          └──────────────────────────┘
```

## AWS Accounts

| Account name      | Purpose                                          | Primary stores         |
|-------------------|--------------------------------------------------|------------------------|
| Ops Suite         | MFE hosting, single-spa root, CDN                | S3, CloudFront         |
| Calm / DHP        | Calm-domain GraphQL APIs and FMLA service        | Aurora PostgreSQL      |
| Crew Core         | Crew Member profile and sensitive crew data      | DynamoDB               |
| Analytics         | Operational analytics warehouse                  | Aurora                 |

## Frontend (MFE)

- Built artefact deployed to S3 in **Ops Suite** AWS account.
- **single-spa** root-config handles mounting and routing across MFEs.
- Two namespaces during rollout: **DEV1** and **Golden DEV**.
- All GraphQL traffic goes through the AppSync endpoint protected by the Lambda
  authorizer.

## API layer (AppSync)

- One AppSync GraphQL API per bounded context (Calm, Crew Member sensitive,
  Crew Schedule).
- Resolvers chosen per the decision matrix in
  `skills/graphql-appsync-standards/references/02-data-source-mapping.md`:
  VTL for thin CRUD on a single data source, Lambda for orchestration / multi-source.
- **No throttling at AppSync level** — scaling is at the Lambda level
  (warm in QA / prod / dev).
- Every API uses a **Lambda authorizer** that:
  1. validates the bearer token,
  2. resolves the user's AD groups,
  3. attaches roles into `$ctx.identity.resolverContext` for downstream resolvers.

## Persistence

- **DynamoDB** in Crew Core for the Crew Member profile (single-table design,
  Global Tables for multi-region, on-demand capacity, point-in-time recovery on).
- **Aurora PostgreSQL** in Calm/DHP for relational FMLA / Calm-domain data
  (Aurora Global Database, RDS Proxy required from Lambda).
- Sensitive PII (e.g., FMLA) is fronted by a dedicated **Crew Member GraphQL API**
  with a stricter authorizer; access from other contexts goes through that API,
  never directly to the DynamoDB table.

## Event-Driven Integrations (Kafka)

| Topic                         | Producer       | Consumer                          | Notes                            |
|-------------------------------|----------------|-----------------------------------|----------------------------------|
| `pdos.employee.profile.v1`    | PDOS           | DynamoDB sync Lambda              | Workday → PDOS → Kafka → Dynamo  |
| `calm.fmla.event.v1`          | Calm           | Workday sync (nightly)            | FMLA Calm → Workday              |
| `css.fly.event.v1`            | CSS            | Crew Schedule consumer            |                                  |
| `css.nonfly.event.v1`         | DMS→Kinesis→EB | Staff-bank non-flies consumer     | CDC pipeline                      |
| `css.staff_bank.cdc.v1`       | DMS→Kinesis→EB | Staff-bank consumer               | CDC pipeline                      |
| `calm.analytics.event.v1`     | Calm           | Analytics CDC consumer            | → Analytics Aurora               |

- **Cross-region replication** via Kafka MirrorMaker 2 (MM2) between US East
  and US West clusters.
- Consumers run on Lambda with SQS or EventBridge in front, with DLQ + idempotency
  per `skills/event-driven-aws-standards/references/03-consumer-patterns.md`.

## Multi-Region Failover

| Layer         | Mechanism                                                   |
|---------------|-------------------------------------------------------------|
| DNS           | Route 53 health-checked failover                            |
| AppSync       | Per-region API (us-east-1 + us-west-2)                      |
| Lambda        | Deployed in both regions; warm in QA/prod/dev               |
| DynamoDB      | Global Tables (multi-region active-active)                  |
| Aurora        | Aurora Global Database (1 primary + 1 secondary region)     |
| Kafka         | MirrorMaker 2 between clusters                              |

The CALM agents must respect this in any IaC they emit (no single-region
hardcoding; see `skills/terraform-aws-standards/references/04-multi-region.md`).

## Operational Analytics Pipeline

```
Calm / Crew APIs  ──▶ Kafka  ──▶ CDC Consumer (Lambda)  ──▶ Analytics AWS acct  ──▶ Analytics Aurora
```

This pipeline is read-only from the analytics side. CALM agents do not write
into Analytics directly; they emit domain events, the CDC consumer projects them.

---

## 2026-05-14 walkthrough — clarifications

The 2026-05-14 architecture walkthrough added these clarifications on top of the
above. Full record: `knowledge/architecture/architecture-walkthrough-2026-05-14.md`.

### Account boundary is hard

The Crew Core <-> Calm/DHP split is enforced at the IAM boundary, not just by
convention. Cross-account access uses:

- Per-API IAM roles assumable only by the owning account's Lambdas.
- VPC endpoints for inter-account AppSync where needed.

The Developer agent must **not** generate IAM policies that grant cross-account
Dynamo/Aurora data-plane access. Cross-account calls go through GraphQL only.

### PII-on-Kafka prohibition

The Kafka topic table above lists topics carrying **identifiers only**. Any new
topic schema that introduces `sensitive` or `regulated` fields is a BLOCKER at
review time. See `knowledge/swa-stack/data-classification.md` for the field list.

### Console-managed resolvers (audit gap)

Some AppSync data-source mappings still live in the AWS console (not codified in
Terraform). The Developer agent must **import** a console resolver into
Terraform if a feature touches it, not silently replace it. The Code Reviewer
flags any new resolver added in console-only form.

### Lambda keep-warm — all environments

Keep-warm is configured in **all four** environments (DEV-1, Golden DEV, QA,
Prod), not Prod-only as some earlier docs implied. Test Engineer test plans
should not include cold-start workarounds for DEV-1.

### Agents target DEV-1 only

See `knowledge/swa-stack/deployment-flow.md` for the full constraint and the
hooks that enforce it.
