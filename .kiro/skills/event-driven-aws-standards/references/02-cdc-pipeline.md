# DMS → Kinesis → EventBridge CDC Pipeline

## Context (per meeting notes)
For CSS staff bank non-flies, data is consumed directly from CSS database tables via:
`DMS task → Kinesis Data Stream → EventBridge → Calm consumer Lambda`

This is in-progress at SWA; the agents must produce IaC and consumers that fit this pipeline.

## Pipeline shape

```
[ CSS database (source) ]
        │
        │  DMS replication task (on-going, full-load + CDC)
        ▼
[ Kinesis Data Stream ]
        │  (per-table stream OR a single stream with table partition key)
        ▼
[ EventBridge Pipe ]
        │  (filter + enrichment + target invocation)
        ▼
[ Calm consumer Lambda (event-driven inbound adapter) ]
        │  applies idempotency · validates schema · projects to Calm domain
        ▼
[ Aurora Calm cluster ]   AND/OR   [ DynamoDB Calm read model ]
```

## Required IaC components

### DMS replication
- Replication instance in a private subnet.
- Source endpoint: CSS DB (read-only credentials in SecretsManager with rotation).
- Target endpoint: Kinesis stream.
- Task type: full-load + CDC.
- Task uses table-mapping rules; only the staff-bank tables are in scope.

### Kinesis Data Stream
- Encryption with customer-managed KMS key.
- Retention 7 days minimum (allows replay during outages).
- Partition key: stable per record (e.g., `staff_bank_id`).
- Number of shards sized for peak; alarm on `IteratorAgeMilliseconds`.

### EventBridge Pipe
- Source: the Kinesis stream.
- Filter: only relevant table-mapped records.
- Enrichment: optional Lambda to normalise into the Calm event contract.
- Target: Calm consumer Lambda OR an EventBridge bus that fans out to multiple consumers.

### Consumer Lambda
- Hexagonal: inbound adapter parses Kinesis event records, calls the use case.
- Idempotency: `event_id` derived from CSS table PK + DMS sequence number.
- batchItemFailures returned to Kinesis on partial failure.
- DLQ configured.

## Replay strategy
- DMS retains full load + CDC; replay by re-pointing the task with a new target stream.
- Kinesis retention provides 7 days; replay via `GetShardIterator` from a checkpointed sequence number.
- The Calm consumer maintains a checkpoint table (DynamoDB) to track per-shard progress.
- Operator command: `tools/cdc-replay.ts --shard-id <id> --since <seq-number>`.

## Multi-region (per meeting notes pattern)
- DMS tasks exist in both regions; only the active region replicates.
- Kinesis streams exist in both regions; cross-region replication is via MM2 for the equivalent Kafka topic if the design uses both.
- On failover, the alternate-region task is enabled.
