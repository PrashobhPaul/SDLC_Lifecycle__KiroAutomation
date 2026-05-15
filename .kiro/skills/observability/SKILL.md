---
name: observability
description: >
  Observability skill for the Kiro CALM construction loop. Activate this skill on every agent turn —
  it provides the canonical logging, tracing, metrics, and dashboard format used across all five agents.
  The skill defines structured AIQ JSONL records, OpenTelemetry trace propagation, sprint scorecard
  aggregation, alerting thresholds, and the integration path with ValidAIte. Always activated.
---

# Observability — Skill

This skill is the contract for how every agent turn becomes traceable, queryable, and reportable.
The 4-pillar commitment requires "Structured JSON logs + OTel traces + metrics + dashboards + alerts".

## How to use this skill

| Concern | Reference |
|---------|-----------|
| Structured AIQ JSONL record format and emission rules | `references/01-aiq-records.md` |
| OpenTelemetry trace propagation across agent handoffs | `references/02-otel-tracing.md` |
| Metrics inventory and Prometheus / CloudWatch mapping | `references/03-metrics.md` |
| Dashboards and what each tile shows | `references/04-dashboards.md` |
| Alert thresholds and escalation | `references/05-alerts.md` |
| ValidAIte sprint aggregation pipeline | `references/06-validaite-pipeline.md` |

## Always-on principles

- One JSONL record per agent turn. No batching. No silent gaps.
- Every record carries `trace_id`, `span_id`, `agent_run_id`, `feature_id`.
- Trace context propagates via the handoff envelope `observability` block.
- Logs never contain PII (names, IDs, dates of birth, schedule details). Use the redactor.
- Metrics are pull-friendly (Prometheus exposition) AND push-friendly (CloudWatch EMF).
- Every alert points to a runbook and an owner.
- Dashboards are versioned in `.kiro/validaite/dashboards/` (Grafana / CloudWatch JSON).
