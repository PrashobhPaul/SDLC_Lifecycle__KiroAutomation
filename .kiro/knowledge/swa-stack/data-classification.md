# SWA Data Classification & PII Handling

Canonical data-classification rules for the CALM Phase-1 pilot, established in the
2026-05-14 CALM architecture walkthrough with the SWA Crew-Ops architecture team.

This file is referenced by:
- `steering/swa-data-scope.md`
- `skills/event-driven-aws-standards/`
- `skills/graphql-appsync-standards/`

---

## Classification tiers

| Tier         | Examples                                                            | Storage allowed                       | Transport allowed                                |
|--------------|---------------------------------------------------------------------|---------------------------------------|--------------------------------------------------|
| `public`     | Flight number, aircraft tail (de-identified)                        | Any AWS service                       | Kafka, AppSync, S3, CloudWatch                   |
| `internal`   | Schedule blocks, crew base codes, duty totals                       | DynamoDB, Aurora, S3 (encrypted)      | Kafka, AppSync, EventBridge                      |
| `sensitive`  | Employee names, employee IDs, contact info                          | DynamoDB (Crew Core)                  | AppSync via Crew-Member API only                 |
| `regulated`  | FMLA records, medical attestation, leave reasons                    | Aurora Postgres (Calm/DHP)            | AppSync via Crew-Member API only                 |

## The Kafka exclusion rule (HARD CONSTRAINT)

**No `sensitive` or `regulated` data may transit Kafka.** Confirmed
2026-05-14 by the architecture team:

- PDOS / Workday / CSS Kafka topics carry **identifiers only** (employee number,
  base, position) — never names, FMLA reasons, or contact info.
- Consumers that need `sensitive` or `regulated` data **must fetch it via the
  Crew-Member GraphQL API** using the identifier from the Kafka event.
- This is enforced at producer-side schema review and at consumer-side static
  scan (Test Engineer covers a static check; Code Reviewer escalates any
  violation to BLOCKER severity).

## The Crew-Member API rule

A dedicated AppSync GraphQL API in the Crew Core account is the **sole source**
of `sensitive` and `regulated` data for downstream consumers. Direct DynamoDB
or Aurora access from outside the owning bounded context is prohibited.

- Authorizer: stricter Lambda authorizer with AD-group scoping per field.
- Field-level access logging: every read of a `sensitive`/`regulated` field is
  written to an audit log in CloudWatch.
- No service-to-service shortcut: even Lambda-to-Lambda inside the same VPC
  must go through the API, not the underlying store.

## Implications for the five agents

| Agent          | Hard rule                                                                                              |
|----------------|--------------------------------------------------------------------------------------------------------|
| feature-builder| If a feature touches `sensitive`/`regulated` data, the handoff envelope must set `data_class` and tag the Crew-Member API dependency. No silent assumption that direct DB access is OK. |
| developer      | When building a new consumer, ALWAYS route `sensitive`/`regulated` reads through the Crew-Member API. Do NOT add the field to the Kafka topic schema even if "it would be convenient". |
| test-engineer  | Add a static test: scan Kafka topic schemas for forbidden field names. Add an integration test that asserts the Crew-Member API is the data source. |
| code-reviewer  | Treat any new Kafka schema field matching the regulated-pattern list as a BLOCKER. Same for direct DB reads of Crew-Core DynamoDB from outside the Crew-Core context. |
| documentation  | The data-classification table in any new doc must match the canonical tiers above; flag drift. |

## Regulated-pattern list (used by reviewer scans)

```yaml
forbidden_in_kafka_schemas:
  - "fmla*"
  - "leave_reason*"
  - "medical*"
  - "ssn"
  - "tax_id"
  - "address"
  - "phone"
  - "email"
  - "dob"
  - "date_of_birth"
```

## See also

- `knowledge/architecture/architecture-walkthrough-2026-05-14.md` — meeting notes.
- `knowledge/security/pii-allow-list.yml` — fine-grained allow-list per field.
- `skills/event-driven-aws-standards/references/03-consumer-patterns.md`.
