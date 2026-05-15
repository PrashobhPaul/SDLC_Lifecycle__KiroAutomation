# Auth Flow: Lambda Authorizer + AD Groups + Roles

## Flow (per CALM architecture meeting notes)

```
[ MFE (Calm UI in Ops Suite) ]
      │
      │  GraphQL request (Authorization: Bearer <token>)
      ▼
[ AWS AppSync ]
      │
      │  invoke authorizer
      ▼
[ Lambda Authorizer (CALM API specific) ]
      │
      │  validate token · check AD groups · resolve roles
      │  return AppSyncAuthorizerResult { isAuthorized, resolverContext, ttlOverride }
      ▼
[ AppSync ]
      │
      │  if isAuthorized=false → return UNAUTHORIZED to MFE
      │  else attach resolverContext to event and invoke resolver
      ▼
[ Lambda resolver | VTL resolver ]
```

## Per-context authorizer
Two authorizers exist (per the meeting notes):
- One for Crew Member API (sensitive data scope).
- One for Calm API (attendance / leave scope).

Both validate against the same AD identity provider but enforce different role ladders.

## resolverContext shape (passed through to handlers)
```ts
type ResolverContext = {
  username: string;
  adGroups: string[];
  roles: ('crew-member' | 'crew-supervisor' | 'crew-admin' | 'fa-instructor' | 'sre' | 'cyber')[];
  ipAddress: string;
  region: 'us-east-1' | 'us-west-2';
  cognitoIdentityId?: string;
};
```

## Authorizer rules (Code Reviewer enforces)
- The authorizer Lambda is in the Ops Suite AWS account, not the API's account (per meeting notes).
- IAM trust policy restricts which AppSync APIs can invoke it (account-scoped).
- The authorizer never returns `isAuthorized=true` with empty `resolverContext.roles`.
- The authorizer caches by `Authorization` header for the configured TTL (default 300s); never include user-specific data in the cache key beyond the token.
- Sensitive-context Queries / Mutations require an `adGroups` membership check inside the resolver, not only at the authorizer.

## Failure modes
- Token expired → `UNAUTHENTICATED` (401-equivalent).
- Token valid, group missing → `FORBIDDEN`.
- Authorizer Lambda timeout → AppSync retries once, then `INTERNAL`.

## Field-level authorisation
- `@auth(requires: [AdGroup!]!)` directive on Query/Mutation. The field-level check happens in the resolver against `resolverContext.adGroups`.
- `@sensitive` fields trigger a second-pass check that the caller's authorizer scope includes the Crew Member API.
