---
name: graphql-appsync-standards
description: >
  GraphQL + AWS AppSync standards skill for the SWA CALM platform. Activate this skill on any task
  that touches schema.graphql, AppSync data-source mapping, Apollo client integration, GraphQL
  codegen, or GraphQL breaking-change review. Encodes SWA's choice of AppSync over API Gateway,
  per-Query/Mutation Lambda or VTL routing, and the Lambda-authorizer auth flow described in the
  CALM architecture meeting notes.
---

# GraphQL + AppSync Standards — Skill

## How to use this skill

| Concern | Reference |
|---------|-----------|
| Schema design and the AppSync model | `references/01-schema-design.md` |
| Data-source mapping: Lambda resolver vs VTL resolver decision | `references/02-data-source-mapping.md` |
| Breaking-change rules + graphql-inspector usage | `references/03-schema-review.md` |
| Apollo client + codegen setup for the React MFE | `references/04-codegen-and-client.md` |
| Auth flow: Lambda authorizer + AD groups + roles | `references/05-auth-flow.md` |

## Always-on principles

- Every CALM API is GraphQL. SWA chose AppSync for its native GraphQL capabilities; do not propose API Gateway/REST.
- Every Query/Mutation has exactly one resolver: Lambda OR VTL. Never both.
- Schema docstrings (`""" ... """`) drive the auto-generated API docs.
- Breaking changes are blocked at G3 unless overridden with justification.
- Sensitive crew-member data goes through the Crew Member Lambda authorizer; never via the Calm context directly.
