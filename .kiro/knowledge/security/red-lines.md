# Red Lines — Hard Refusal Categories

CALM agents must refuse, halt, and emit a refusal record when any of the
following are requested or detected. The refusal preserves context so the user
can retry with corrected inputs.

## Code red lines

- Producing code that hardcodes secrets, AWS access keys, or production endpoints.
- Producing code that disables TLS, certificate verification, or AWS SDK
  signature validation.
- Producing code that bypasses the Lambda authorizer or strips the
  `$ctx.identity.resolverContext` checks.
- Producing code that swallows exceptions silently (`catch (e) {}`).

## Data red lines

- Emitting unredacted PII into AIQ records, logs, or handoff envelopes.
- Reading or referencing crew member sensitive data outside the dedicated
  Crew Member GraphQL API.
- Cross-context direct DB access.

## Architectural red lines

- Hardcoding region, account ID, or ARN in Terraform (BE Lambda audit rule 13,
  CRITICAL).
- IAM policies with `Action: "*"` or `Resource: "*"` on a service principal that
  fronts user data (BE rule 61).
- Unbounded `Promise.all` over an external API or DB (BE rule 21).
- Domain-layer files importing `@aws-sdk/*` (BE rule 10, CRITICAL).

## Process red lines

- Pushing to a protected branch without G9 approval recorded.
- Running `terraform apply` against any environment from inside an agent run.
- Auto-resolving an open Question-Back without the user marking it `answered`.
- Skipping any HITL gate that the gate registry marks as required for the layer.

## Behaviour on red-line match

1. Halt the current turn.
2. Emit a refusal record (schema: `guardrails-io-sanity` skill, reference 07).
3. Surface the matched rule and the corrective action to the user.
4. Resume only when the user submits a corrected input that no longer matches.
