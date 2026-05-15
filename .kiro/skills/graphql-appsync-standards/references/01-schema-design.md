# Schema Design

## File layout
```
schema/
├── schema.graphql          # root, imports per bounded context
├── crew-member.graphql     # Crew Member bounded context types
├── calm.graphql            # Calm bounded context types
├── crew-schedule.graphql   # Crew Schedule bounded context types
├── shared.graphql          # scalars, enums, common types
└── directives.graphql      # custom directives (e.g. @auth, @sensitive)
```

## Naming
- Types: `PascalCase` — `Pilot`, `LeaveRequest`, `RestViolation`.
- Fields: `camelCase` — `crewId`, `restRemainingMinutes`.
- Enums: `PascalCase` values UPPER_SNAKE — `LeaveStatus { OPEN, APPROVED, REJECTED }`.
- Inputs: suffix `Input` — `CreateLeaveRequestInput`.
- Payloads: suffix `Payload` — `CreateLeaveRequestPayload` (for mutation responses).

## Required directives (enforced by the linter)
- `@auth(requires: [AdGroup!]!)` on every Query/Mutation/Subscription.
- `@sensitive` on fields containing PII (SSN, passport, full DOB, full home address) — these
  resolve via the Crew Member Lambda authorizer scope only.

## Pagination
- Connection-style pagination only. No offset-based pagination.
- `Connection`, `Edge`, `PageInfo` follow Relay style.

## Errors
- Errors flow through GraphQL error extensions, not field nullability tricks.
- `extensions.code` from a fixed enum: `UNAUTHORIZED | FORBIDDEN | NOT_FOUND | VALIDATION | INTERNAL`.
- Internal errors never include stack traces (per BE rule 23).

## Versioning
- Additive changes preferred. Removed/renamed fields are deprecation cycles:
  1. Mark `@deprecated(reason: "Use X")`.
  2. Wait at least one full release cycle.
  3. Remove only after consumer audit.

## Subscriptions
- AppSync subscriptions only when there is a real-time requirement.
- Each subscription names the mutation that triggers it via `@aws_subscribe(mutations: ["..."])`.
