# Bounded Contexts

| Context        | AWS Account     | Primary Store     | Notes                                  |
|----------------|-----------------|-------------------|----------------------------------------|
| Crew Member    | Crew Core       | DynamoDB          | PII; fronted by sensitive-info GraphQL |
| Calm           | Calm / DHP      | Aurora PostgreSQL | FMLA + Calm-domain entities            |
| Crew Schedule  | Calm / DHP      | Aurora PostgreSQL | Pairings, rest periods                 |
| Sensitive Info | Crew Core       | DynamoDB (read)   | Lambda-only access via dedicated API   |
| Analytics      | Analytics       | Aurora            | Read-only projection of domain events  |

## Cross-context rules

- **No direct DB access across contexts.** Reads go through the owning GraphQL API.
- **No shared write tables across contexts.** Each domain owns its store.
- **Sensitive PII never leaves Crew Core** except via the sensitive-info GraphQL API,
  whose authorizer enforces stricter AD-group checks.
- **Events are the only sanctioned cross-context coupling** (Kafka topics in
  `aws-stack.md`).
