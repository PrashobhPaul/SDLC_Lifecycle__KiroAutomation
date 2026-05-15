# Pilot Feature — Crew Rest Violation Alert (FAA Part 117)

This is the seed feature used across the CALM Phase-1 demo. It exercises every
layer (react-mfe → graphql-schema → node-resolver → vtl-resolver → terraform-iac)
and every HITL gate.

## One-line description

Detect and surface crew rest violations against FAA Part 117 minimums; alert
crew schedulers in the Calm MFE; record audit trail.

## Domain inputs

- Pairing data from CSS (Kafka topic `css.fly.event.v1`).
- Crew Member profile from PDOS (`pdos.employee.profile.v1`) — joined via
  the dedicated Crew Member GraphQL API.
- Calm-domain rest-period rules (Aurora PostgreSQL).

## Output surfaces

- AppSync GraphQL `restViolations(pairingId, crewId)` query.
- Subscription `onRestViolationAlert(crewId)` for real-time MFE updates.
- React MFE component `RestViolationBadge` and modal `RestViolationDetail`.

## Why this pilot

It exercises:
- VTL resolver (DynamoDB read for crew profile),
- Lambda resolver (Aurora + DynamoDB orchestration),
- Kafka consumer (pairing event ingest),
- Multi-region failover (alert must survive a region cutover),
- All four pillars (governance, observability, guardrails, async orchestration),
- All five agents and all nine HITL gates.

## Phase-1 success criteria for the pilot

| Metric                              | Target |
|-------------------------------------|--------|
| Compilation pass on first build     | ≥ 98%  |
| Pattern adherence (UI + BE)         | ≥ 4.0 / 5.0 |
| Hallucination rate                  | ≤ 2%   |
| Handshake fidelity (envelopes)      | ≥ 99%  |
| Coverage on generated code          | ≥ 80%  |
| Reviewer false-positive rate        | ≤ 15%  |
| First-pass acceptance by SME        | ≥ 60%  |
| Token reduction vs v1 baseline      | ≥ 30%  |
| Human-vs-judge correlation          | ≥ 0.7  |
| HITL gate compliance                | 100%   |
