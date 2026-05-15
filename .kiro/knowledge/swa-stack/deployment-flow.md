# SWA Deployment Flow — DEV-1 → Golden DEV → QA → Prod

Canonical environment promotion path for CALM, confirmed in the 2026-05-14
architecture walkthrough.

## The path

```
   ┌─────────┐    ┌──────────────┐    ┌─────┐    ┌──────┐
   │  DEV-1  │───▶│  Golden DEV  │───▶│ QA  │───▶│ Prod │
   └─────────┘    └──────────────┘    └─────┘    └──────┘
       ▲
       │
   AI agents may ONLY target DEV-1
   (no exceptions, no overrides)
```

## Environment characteristics

| Env         | Purpose                                                | Lambda keep-warm | Aurora Global DB         | Who may deploy            |
|-------------|--------------------------------------------------------|------------------|--------------------------|---------------------------|
| DEV-1       | Agent-built feature branches, integration smoke        | yes              | single region            | AI agents, developers     |
| Golden DEV  | Integration baseline, all agent work converges here    | yes              | single region            | developers (merge)        |
| QA          | Manual / regression test, performance validation       | yes              | single region (replica)  | release manager           |
| Prod        | Customer traffic, multi-region active-active           | yes              | US-East + US-West Global | release manager           |

**Lambda keep-warm** is enabled in **all four** environments per the architecture
team — including DEV-1 — so cold-start latency does not skew agent-built tests.

## The agent constraint (HARD)

**AI agents may only target DEV-1.** This is a non-negotiable security boundary:

- Terraform plans produced by the Developer agent are scoped to the DEV-1
  workspace via state-bucket convention and provider configuration.
- `terraform apply` is never invoked by the Developer agent in any environment;
  apply is a human action via the SWA GitLab CI pipeline.
- Promotion from DEV-1 → Golden DEV is a manual GitLab merge by a human reviewer.
- Promotion past Golden DEV is a release-manager action.

Hooks that enforce this:
- `hooks/pre-merge/terraform-plan-required.json` — blocks merges without a plan.
- `hooks/agent-turn/pre-turn-allow-list-check.json` — blocks deny-listed targets
  (`terraform-apply`, any non-DEV-1 environment).

## Multi-region story (Prod only)

- Active-active across `us-east-1` and `us-east-2` (treated as East/West for
  Crew-Ops purposes; physical region names confirmed by Prashanth).
- Route53 weighted records for cross-region traffic distribution.
- Aurora Global Database for sub-second failover RPO; Crew-Member DynamoDB uses
  Global Tables (eventual consistency between regions, last-writer-wins).
- Failover is manual at the Route53 level; nothing is auto-promoted.

## CSS CDC pipeline (in progress)

A new CDC pipeline is being built to ingest CSS staff-bank changes:

```
   CSS DB ──DMS──▶ Kinesis Data Streams ──▶ EventBridge ──▶ Lambda consumer
```

- Status as of 2026-05-14: in-progress, not yet in DEV-1.
- Agents should NOT generate consumers for `css.staff_bank.cdc.v1` until the
  schema is finalised; the Feature Builder is to halt and ask if a feature
  touches this topic.

## Audit gaps (called out 2026-05-14)

- Some AppSync data-source mappings still live in the AWS console (not codified
  in Terraform yet). Agents must NOT modify console-managed resolvers; the
  Developer agent is to add a Terraform import if a console resolver is in
  the feature scope, NOT replace it silently.
- Owners (F2 follow-up): Prashanth to share the Lambda-authorizer repo URL and
  the inventory of console-managed resolvers.

## See also

- `knowledge/swa-stack/aws-stack.md` — full stack reference.
- `knowledge/architecture/architecture-walkthrough-2026-05-14.md` — meeting record.
