# Data-Source Mapping: Lambda vs VTL

## Decision matrix

| Operation | Resolver kind | Why |
|-----------|--------------|-----|
| Single-item DynamoDB GetItem / PutItem / UpdateItem | VTL | No transformation needed; lower cost, lower cold-start |
| DynamoDB Query with simple key conditions | VTL | Same |
| Multi-step business logic (Calm leave eligibility) | Lambda | Requires use cases, branching, external calls |
| Cross-context join (Crew Member + CSS data) | Lambda | Aggregation, must use ports |
| Sensitive crew-member field resolution (SSN, passport) | Lambda | Calls Crew Member dedicated GraphQL API; auth-sensitive |
| FMLA → Workday write-through | Lambda | Calls Kafka publisher + API; idempotency required |
| Operational analytics queries (read-only Aurora) | Lambda | RDS connection pooling via RDS Proxy |
| Pipeline resolvers (auth check then fetch) | VTL pipeline | Composable VTL templates |

## Mapping registry
Every Query/Mutation/Subscription is mapped in `terraform/appsync/<api>/resolvers.tf`:

```hcl
resource "aws_appsync_resolver" "queryPilotById" {
  api_id      = aws_appsync_graphql_api.calm.id
  type        = "Query"
  field       = "pilotById"
  data_source = aws_appsync_datasource.crew_member_lambda.name  # or .dynamodb_pilots.name
  kind        = "UNIT"  # or "PIPELINE"
  request_template  = file("${path.module}/vtl/Query.pilotById.req.vtl")
  response_template = file("${path.module}/vtl/Query.pilotById.res.vtl")
}
```

## Per-context bounding
- `crew_member_lambda` data source lives in the Crew Core AWS account; cross-account invocation by AppSync is configured in the resolver IAM.
- `dynamodb_pilots` lives in Crew Core; Calm context never reads DynamoDB tables it does not own.
- `aurora_calm` lives in the DHP/Calm AWS account; only Lambda resolvers under that account can talk to it (RDS Proxy).

## Throttling
Per the meeting notes: scaling is at the Lambda level; no throttling at AppSync. The Developer agent does not propose AppSync-level rate limits unless asked.

## Cold start
Per meeting notes: Lambdas are warmed in QA / prod / dev. The Developer agent does not propose provisioned concurrency unless the latency budget for a specific resolver demands it; if so, document in the IaC plan.
