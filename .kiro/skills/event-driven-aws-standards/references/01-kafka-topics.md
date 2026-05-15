# Kafka Topics & Contracts

## Topic catalogue (per meeting notes)

| Topic | Producer | Consumer | Direction | Purpose |
|-------|---------|----------|-----------|---------|
| `pdos.employee.profile.v1` | PDOS | Crew Member, Calm | inbound | Workday → PDOS → Kafka pipeline; populates DynamoDB pilot profile |
| `calm.fmla.event.v1` | Calm | Workday consumer | outbound | FMLA events sent nightly |
| `css.fly.event.v1` | CSS | Calm | inbound | Fly events |
| `css.nonfly.event.v1` | CSS | Calm | inbound | Non-fly events |
| `css.staff_bank.cdc.v1` | DMS→Kinesis→EventBridge bridge | Calm | inbound | CSS staff bank non-flies (CDC, in-progress per meeting notes) |
| `calm.analytics.event.v1` | Calm | Analytics CDC consumer | outbound | Operational analytics → Aurora analytics |

Naming: `<source>.<entity>.<flavor>.v<n>`. Major-version bump on breaking schema change.

## Contract (Avro / JSON Schema)

Stored in `.kiro/handoff/event-schemas/` and registered in Schema Registry at deploy time.

```json
{
  "namespace": "calm.events",
  "type": "record",
  "name": "FmlaEvent",
  "version": 1,
  "fields": [
    { "name": "event_id", "type": "string" },
    { "name": "schema_version", "type": "int" },
    { "name": "occurred_at", "type": { "type": "string", "logicalType": "iso-datetime" } },
    { "name": "crew_id_hash", "type": "string" },
    { "name": "fmla_case", "type": { "type": "record", "name": "FmlaCase", "fields": [...] } },
    { "name": "trace_id", "type": ["null", "string"], "default": null }
  ]
}
```

### Required event fields
- `event_id` (UUID v4) — for idempotency.
- `schema_version` — for routing logic.
- `occurred_at` — RFC 3339.
- `trace_id` — for distributed tracing.
- Domain identifiers HASHED if they are PII (`crew_id_hash`, not `crew_id` raw).

## Cross-region (MM2)
- Every `inbound` topic is mirrored from us-east-1 to us-west-2 with the same name.
- Every `outbound` topic is also mirrored so consumers in either region see the same stream.
- MM2 offset-sync configured; on failover, SRE resets consumer offsets per the playbook.

## Producer rules
- Idempotent producer enabled (`enable.idempotence=true`).
- `acks=all`.
- Retries with exponential backoff.
- Schema validation against the registered schema BEFORE publish.

## Compaction & retention
- Profile/state topics: log-compacted, infinite retention with tombstones.
- Event topics: 7-day retention default; longer with documented justification.
- Analytics-source topics: 14-day retention, downstream consumes within 24h.
