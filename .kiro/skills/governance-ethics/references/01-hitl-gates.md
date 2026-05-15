# 9 Mandatory HITL Gates

Phase 1 commitment: every gate fires when its trigger condition is met.
HITL audit shows 100% compliance across the chain.

## Inventory

| Gate | Name | Owner | Trigger | Blocking | Evidence |
|------|------|-------|---------|----------|----------|
| G1 | scope-approval | feature-builder | After Epic/Story decomposition | Yes | scope.in_scope, scope.out_of_scope, AC |
| G2 | plan-approval | developer | After codebase analysis + clarification | Yes | files-table, steps, integration, risks |
| G3 | schema-change-approval | developer | Any GraphQL schema change | Yes | schema diff, breaking-check, codegen output |
| G4 | iac-change-approval | developer | Any Terraform change | Yes | terraform plan, tfsec, IAM diff |
| G5 | test-strategy-approval | test-engineer | Before generating tests | Yes | pyramid, edge cases, factories, coverage target |
| G6 | review-blocker-override | code-reviewer | Override of any blocker finding | Yes | finding-id, rationale, approver, expiry |
| G7 | self-correction-approval | developer | Mechanical fix after build verify fails | Yes | proposed diff, before/after verification |
| G8 | doc-stale-decision | documentation | Stale-doc detection match | No | doc path, drift, proposed update |
| G9 | pre-merge-approval | code-reviewer | Final pre-merge gate | Yes | gate statuses, CI, coverage delta, security |

## Hard rules

1. The literal token `approve` (case-insensitive) advances a gate. `ok`, `yes`, `looks good`, `lgtm` do **not**.
2. `reject` halts. `modify <text>` applies the change and re-prompts the gate.
3. Every gate emits an AIQ record with `hitl.gate_fired = G<n>`, `hitl.outcome ∈ {approved, rejected, modified, n/a}`.
4. Compliance metric: `fired_gates / required_gates per pilot run`. Target: 100%.

## Override expiry

Overrides through G6 carry an explicit expiry (default 7 days). After expiry the finding re-blocks the next pre-merge.
The override record is written to `.kiro/handoff/<feature-id>/blocker-override.md` and surfaced in the sprint scorecard.
