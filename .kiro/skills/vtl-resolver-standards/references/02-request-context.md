# Request Context (`$ctx`) Shape

## Identity (from Lambda authorizer)
```
$ctx.identity = {
  "username": String,
  "claims":   { ...JWT claims },
  "resolverContext": {
    "adGroups":  [String],
    "roles":     [String],
    "ipAddress": String,
    "region":    String
  }
}
```

## Arguments
`$ctx.args` — the GraphQL field arguments.
`$ctx.source` — for nested fields, the parent object.

## Stash
`$ctx.stash` — pipeline-resolver scratch space carried across functions.

## Result
`$ctx.result` (response template) — output from the data source.
`$ctx.prev.result` (pipeline) — output of the previous function.

## Error
`$ctx.error.message`, `$ctx.error.type` — populate on data-source error.

## Util helpers (always-on)
- `$util.toJson(...)` — never use `$util.toJson($ctx.result.items)` if items can include sensitive fields.
- `$util.dynamodb.toDynamoDBJson(...)` — for DynamoDB request templates.
- `$util.error(message, type)` — raise an error with a typed code (`NOT_FOUND`, `FORBIDDEN`, `VALIDATION`).
- `$util.unauthorized()` — raise an `UNAUTHORIZED` error.
- `$util.time.nowISO8601()` — RFC 3339 timestamp.
- `$util.autoId()` — UUID v4 generator.

## Anti-patterns
- Reading `$ctx.identity.claims` directly for auth — use `resolverContext.adGroups` populated by the Lambda authorizer.
- Concatenating user input into expression strings — always use `expressionValues`.
- Returning `$ctx.error` content to the client — strip stack traces.
