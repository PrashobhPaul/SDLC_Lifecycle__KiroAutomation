# Multi-Region Failover — Reference

| Concern                     | Implementation                                                  |
|-----------------------------|-----------------------------------------------------------------|
| Region pair                 | `us-east-1` (primary) + `us-west-2` (DR)                        |
| DNS failover                | Route 53 health-checked records, TTL 30 s                       |
| AppSync                     | Independent API per region; client uses Route 53 endpoint       |
| Lambda                      | Deployed and warmed in both regions                             |
| DynamoDB                    | Global Tables; writes go to local region, replicate active-active|
| Aurora PostgreSQL           | Aurora Global Database, 1 writer + read replicas in DR region   |
| Aurora failover RPO         | < 1 second                                                      |
| Aurora failover RTO         | < 1 minute (managed failover)                                   |
| Kafka                       | MirrorMaker 2 between regional clusters                         |
| Secrets                     | Replicated AWS Secrets Manager secret across both regions       |
| KMS                         | Multi-region keys for encryption-at-rest                        |

## Agent obligations

| Agent              | Obligation                                                                |
|--------------------|---------------------------------------------------------------------------|
| Developer          | Emit Terraform parameterised on region; provider per region; no hardcode  |
| Developer          | Resolvers must not assume region; read region from context                |
| Test Engineer      | Include a single failover-simulation test per pilot                       |
| Code Reviewer      | Block merge on region/account hardcoding (BE rule 13 CRITICAL)            |
| Documentation      | Emit a multi-region diagram (Mermaid + drawio) in the feature ADR         |

## What is explicitly NOT in scope for Phase-1

- Live MM2 fail-back drills (out of scope; documented for Phase-2).
- Cross-region writes to a single Aurora primary (use the standard active/passive).
