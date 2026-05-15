# Multi-Region Failover

## Architecture (per CALM meeting notes)

```
                         ┌──────────────────┐
                         │    Route53 DNS   │
                         │ health-check     │
                         │ failover policy  │
                         └────────┬─────────┘
                                  │
                ┌─────────────────┴─────────────────┐
                ▼                                   ▼
       ┌────────────────┐                  ┌────────────────┐
       │   us-east-1    │                  │   us-west-2    │
       │   PRIMARY      │ ◀── failover ──▶ │  STANDBY       │
       │                │                  │                │
       │  AppSync API   │                  │  AppSync API   │
       │  Lambdas       │                  │  Lambdas       │
       │  DynamoDB GT   │ ←  Global ──→    │  DynamoDB GT   │
       │  Aurora Global │ ←  replica ──→   │  Aurora Global │
       │  Kafka MM2     │ ←  mirror ───→   │  Kafka MM2     │
       └────────────────┘                  └────────────────┘
```

## Required resources

### Route53
- Health check on the primary AppSync endpoint per region.
- Failover routing policy: PRIMARY → us-east-1, SECONDARY → us-west-2.
- TTL 30s during a sprint, 60s steady-state.

### DynamoDB Global Tables
- Every CALM table that holds active data is configured as a Global Table.
- Replicas in us-east-1 and us-west-2.
- Eventual consistency across regions; the Developer agent flags any pattern that requires strong consistency cross-region (the answer is usually "redesign").

### Aurora Global Database
- Calm bounded context Aurora cluster has a secondary region cluster (us-west-2).
- RTO < 1 minute, RPO < 1 second per Aurora Global SLA.
- Read-only access in standby region; writes only after promotion.

### Kafka MirrorMaker2
- Replicates topics across the two regions.
- Consumer offsets replicated; reset is an SRE-driven action during failover.
- Non-replicated topics explicitly tagged `replication: none`.

### Lambda
- Lambdas exist in both regions; deployed via the same Terraform module with `provider = aws.us_east_1` or `provider = aws.us_west_2`.
- Warm-keeper schedule fires in both regions (per meeting notes; warm in QA / prod / dev).
- Concurrency reserved per region.

### S3 buckets
- Cross-region replication on critical buckets (state, MFE bundles, validaite archives).

## Failover procedure (SRE-owned, captured here so the agents do not propose anything contrary)
1. Route53 health check fails on us-east-1 endpoint.
2. Route53 swaps to us-west-2 transparently for clients.
3. Aurora Global secondary is promoted (manual + automation).
4. Kafka consumers/producers cut over (offsets reset if needed).
5. Validation playbook executed.

## What the Developer agent must not propose
- Region-pinned constants in code.
- Endpoints hardcoded to us-east-1.
- Dead-letter routing that only exists in one region.
- DynamoDB tables that are not Global Tables for any data the Calm UI relies on.
- Aurora clusters without a Global secondary for the Calm context.
