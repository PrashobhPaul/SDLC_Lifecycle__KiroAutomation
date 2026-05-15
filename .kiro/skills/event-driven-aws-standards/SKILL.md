---
name: event-driven-aws-standards
description: >
  Event-driven AWS standards skill for the SWA CALM platform. Activate this skill on any task that
  involves Kafka topic design, DynamoDB Streams, EventBridge, SQS, SNS, the DMS-to-Kinesis-to-EventBridge
  CDC pipeline (per CSS integration in meeting notes), or any cross-context event flow between
  Workday, PDOS, CSS, Crew Member, and Calm. Encodes idempotency, batchItemFailures, DLQ, replay,
  and multi-region MM2 mirroring.
---

# Event-Driven AWS Standards — Skill

## How to use this skill

| Concern | Reference |
|---------|-----------|
| Kafka topic design and contract | `references/01-kafka-topics.md` |
| DMS → Kinesis → EventBridge CDC pipeline (CSS staff bank non-flies) | `references/02-cdc-pipeline.md` |
| SQS / SNS / EventBridge consumer patterns | `references/03-consumer-patterns.md` |
| Testing event consumers (idempotency, ordering, replay) | `references/04-testing-consumers.md` |

## Always-on principles

- Every consumer is idempotent (BE rule 25 — MAJOR if missing).
- SQS consumers always implement `batchItemFailures` (BE rule 18).
- Every async producer publishes to a DLQ on failure.
- Schema versioning for every event (BE checklist M).
- Multi-region: every cross-context topic is mirrored via MM2.
