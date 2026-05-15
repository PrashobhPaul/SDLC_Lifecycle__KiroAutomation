---
name: async-orchestration
description: >
  Async orchestration skill providing durable queues, retries with exponential backoff, fallback modes,
  DLQ handling, replay safety, and idempotency-aware scheduling for the Kiro CALM construction loop.
  Activate this skill on every external mutation (Jira write-back, Figma comment, GitLab MR), every
  inter-agent handoff that may be re-driven, and every tool call that can fail transiently. Always
  activated for the five CALM agents.
---

# Async Orchestration — Skill

This skill defines how every fallible action becomes durable and replay-safe.

## How to use this skill

| Concern | Reference |
|---------|-----------|
| Durable queue model and job envelope | `references/01-queue-model.md` |
| Retry policy: 3× exponential backoff | `references/02-retry-policy.md` |
| Fallback modes per action class | `references/03-fallback-modes.md` |
| DLQ handling and replay tools | `references/04-dlq-replay.md` |
| Idempotency integration with the guardrail key | `references/05-idempotency-integration.md` |

## Always-on rules

1. Every external mutation goes through the orchestrator — no direct tool call without a queued job.
2. Every job carries an idempotency key (see `guardrails-io-sanity/references/06-idempotency.md`).
3. Retries are 3× exponential backoff: 1s, 4s, 16s.
4. Fallback path defined per action class: write a local artifact and a deferred job pointer.
5. After 3 failed attempts, route to DLQ at `.kiro/validaite/telemetry/dlq.jsonl`.
6. Replay is a manual operator action via `tools/dlq-replay.ts` after triage.
