---
name: vtl-resolver-standards
description: >
  AppSync VTL resolver standards skill for the SWA CALM platform. Activate this skill on any task
  that touches request/response mapping templates (.vtl), pipeline resolvers, AppSync simulator
  testing, or the request-context shape used by Lambda authorizers. Encodes the mapping templates
  for GetItem, PutItem, Query, UpdateItem, Pipeline functions, and the sensitive-info pattern used
  by the Crew Member dedicated GraphQL API.
---

# VTL Resolver Standards — Skill

## How to use this skill

| Concern | Reference |
|---------|-----------|
| Mapping template patterns (Get/Put/Query/Update/Pipeline) | `references/01-mapping-templates.md` |
| Request context shape (from Lambda authorizer) | `references/02-request-context.md` |
| Testing on the AppSync simulator | `references/03-testing-vtl.md` |
| Review checklist for VTL changes | `references/04-review-checklist.md` |

## Always-on principles

- VTL is preferred for direct DynamoDB single-item operations and simple queries.
- VTL is NOT used for multi-step business logic — that goes through a Lambda resolver.
- Every template includes an `$util.error(...)` path on the failure branch.
- Sensitive-info fields (`@sensitive`) resolve via the Crew Member context only; VTL never returns them from the Calm context.
- Every VTL template has a paired golden test on the AppSync simulator.
