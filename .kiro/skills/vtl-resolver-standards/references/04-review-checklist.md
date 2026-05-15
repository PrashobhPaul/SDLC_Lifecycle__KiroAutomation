# VTL Review Checklist

## Per template

### Request template
- [ ] `version` matches the AppSync VTL version expected by the data source
- [ ] No user input concatenated into expressions; uses `expressionValues`
- [ ] DynamoDB key shape matches the table (pk/sk pattern)
- [ ] `consistentRead` set explicitly when relevant
- [ ] GSI name is correct (`index: "byBase"`) and matches Terraform
- [ ] Pagination via `limit` + `nextToken`; never unbounded
- [ ] No hardcoded ids, region names, or env-specific strings

### Response template
- [ ] `$ctx.error` branch handled with `$util.error`
- [ ] `null` result branch handled (NOT_FOUND error type)
- [ ] No stack-trace leakage to the client
- [ ] No raw return of sensitive fields from a Calm-context resolver
- [ ] Output JSON shape matches the GraphQL return type

### Pipeline-specific
- [ ] `$ctx.stash` keys are documented
- [ ] Each function has its own request/response templates
- [ ] Auth-check function fires `$util.unauthorized()` correctly
- [ ] Order of functions enforces auth-before-data

## Per data-source mapping (in Terraform)
- [ ] Resolver kind matches the template (UNIT vs PIPELINE)
- [ ] IAM role on the resolver allows only the operations actually used (no `dynamodb:*`)
- [ ] Cross-account invocation policy correct when the data source is in another account

## Per change set (Code Reviewer)
- [ ] Tests added for new templates (cases 1–4 from `03-testing-vtl.md`)
- [ ] Existing tests still pass
- [ ] Schema field on which the resolver attaches still exists
- [ ] If the schema changed, breaking-change report attached (see `graphql-appsync-standards/references/03-schema-review.md`)

## Severity mapping
| Finding | Severity |
|---------|----------|
| User input concatenated into expression strings | CRITICAL (injection) |
| Sensitive field returned from Calm-context VTL | CRITICAL |
| Stack trace leaked through `$ctx.error` | CRITICAL |
| Missing `$util.error` on failure branch | MAJOR |
| Missing test for new template | MAJOR |
| Hardcoded id / region | MAJOR |
| Unbounded query (no `limit`) | MAJOR |
| `consistentRead` not set explicitly when correctness depends on it | WARN |
