# VTL Mapping Templates

## GetItem (Query.pilotById)

```vtl
## Query.pilotById.req.vtl
#set($pilotId = $util.dynamodb.toStringJson($ctx.args.pilotId))
{
  "version": "2018-05-29",
  "operation": "GetItem",
  "key": {
    "pk": $util.dynamodb.toDynamoDBJson("PILOT#$ctx.args.pilotId"),
    "sk": $util.dynamodb.toDynamoDBJson("PROFILE")
  },
  "consistentRead": false
}
```

```vtl
## Query.pilotById.res.vtl
#if($ctx.error)
  $util.error($ctx.error.message, $ctx.error.type)
#end
#if($util.isNull($ctx.result))
  $util.error("Pilot not found", "NOT_FOUND")
#end
$util.toJson($ctx.result)
```

## PutItem (Mutation.upsertPilot)

```vtl
## Mutation.upsertPilot.req.vtl
#set($now = $util.time.nowISO8601())
{
  "version": "2018-05-29",
  "operation": "PutItem",
  "key": {
    "pk": $util.dynamodb.toDynamoDBJson("PILOT#$ctx.args.input.pilotId"),
    "sk": $util.dynamodb.toDynamoDBJson("PROFILE")
  },
  "attributeValues": $util.dynamodb.toMapValuesJson({
    "pilotId":   "$ctx.args.input.pilotId",
    "name":      "$ctx.args.input.name",
    "base":      "$ctx.args.input.base",
    "updatedAt": "$now",
    "updatedBy": "$ctx.identity.username"
  }),
  "condition": {
    "expression": "attribute_not_exists(pk) OR updatedBy = :u",
    "expressionValues": {
      ":u": $util.dynamodb.toDynamoDBJson($ctx.identity.username)
    }
  }
}
```

## Query (Query.pilotsByBase, with GSI)

```vtl
## Query.pilotsByBase.req.vtl
{
  "version": "2018-05-29",
  "operation": "Query",
  "index": "byBase",
  "query": {
    "expression": "#base = :base",
    "expressionNames": { "#base": "base" },
    "expressionValues": {
      ":base": $util.dynamodb.toDynamoDBJson($ctx.args.base)
    }
  },
  "limit": $util.defaultIfNull($ctx.args.limit, 50),
  #if($ctx.args.nextToken)
    "nextToken": "$ctx.args.nextToken"
  #end
}
```

```vtl
## Query.pilotsByBase.res.vtl
#if($ctx.error)
  $util.error($ctx.error.message, $ctx.error.type)
#end
{
  "items":     $util.toJson($ctx.result.items),
  "nextToken": $util.toJson($util.defaultIfNull($ctx.result.nextToken, null))
}
```

## UpdateItem (Mutation.updatePilotStatus)

```vtl
## Mutation.updatePilotStatus.req.vtl
#set($now = $util.time.nowISO8601())
{
  "version": "2018-05-29",
  "operation": "UpdateItem",
  "key": {
    "pk": $util.dynamodb.toDynamoDBJson("PILOT#$ctx.args.pilotId"),
    "sk": $util.dynamodb.toDynamoDBJson("PROFILE")
  },
  "update": {
    "expression": "SET #s = :s, updatedAt = :t, updatedBy = :u",
    "expressionNames": { "#s": "status" },
    "expressionValues": {
      ":s": $util.dynamodb.toDynamoDBJson($ctx.args.status),
      ":t": $util.dynamodb.toDynamoDBJson($now),
      ":u": $util.dynamodb.toDynamoDBJson($ctx.identity.username)
    }
  },
  "condition": {
    "expression": "attribute_exists(pk)"
  }
}
```

## Pipeline (auth check then fetch)

```vtl
## fn-authCheck.req.vtl
{
  "version": "2018-05-29",
  "operation": "Invoke",
  "payload": {
    "action": "authorize",
    "resource": "pilot:${ctx.args.pilotId}",
    "identity": $util.toJson($ctx.identity)
  }
}
```
```vtl
## fn-authCheck.res.vtl
#if($ctx.result.allowed != true)
  $util.unauthorized()
#end
$util.toJson({})
```

```vtl
## fn-fetchPilot.req.vtl   (UNIT against DynamoDB)
{
  "version": "2018-05-29",
  "operation": "GetItem",
  "key": {
    "pk": $util.dynamodb.toDynamoDBJson("PILOT#${ctx.stash.pilotId}"),
    "sk": $util.dynamodb.toDynamoDBJson("PROFILE")
  }
}
```

## Sensitive-info pattern (Crew Member API only)

VTL never returns SSN, passport, full DOB, or full home address.
Those fields are resolved by a Lambda in the Crew Member bounded context
that includes a second-pass authorizer check on the caller's `adGroups`.

A Calm-context VTL that touches a sensitive type fetches by id only and
resolves the sensitive fields with a Lambda data source pointed at the Crew Member API.
