# Apollo Client + Codegen

## codegen configuration

`codegen.yml` at project root:
```yaml
schema: ./schema/schema.graphql
documents: 'src/**/*.{ts,tsx,graphql}'
generates:
  src/__generated__/graphql.ts:
    plugins:
      - typescript
      - typescript-operations
      - typescript-react-apollo
    config:
      reactApolloVersion: 3
      withHooks: true
      withHOC: false
      withComponent: false
      avoidOptionals:
        field: true
      enumsAsTypes: true
      scalars:
        AWSDateTime: string
        AWSDate: string
        AWSJSON: 'Record<string, unknown>'
```

Hook: `onFileSave` of `schema.graphql` regenerates `src/__generated__/graphql.ts` and surfaces any
breaking changes via `graphql-inspector` (see `03-schema-review.md`).

## Apollo client setup
- HTTP link points to the AppSync endpoint.
- Auth link injects the bearer token from the Lambda authorizer flow.
- Error link handles `UNAUTHENTICATED` by triggering Ops Suite re-auth.
- In-memory cache typed via the generated `TypePolicies`.

## Operation rules
- `useQuery` / `useMutation` / `useLazyQuery` MUST use generated hooks. Manual hand-rolled types are flagged.
- Every operation handles `loading` and `error` (per UI rule 27/28; MAJOR per callsite if missing).
- Every mutation that mutates visible UI provides an optimistic response OR a documented cache-update strategy.
- No string-template interpolation into `gql` (per UI checklist V; MAJOR — convert to fragments).

## Fragments
- One fragment per type per concern (e.g., `PilotProfile`, `PilotSchedule`).
- Co-located with the consumer when feature-specific; under `src/graphql/fragments/` when shared.
- All fragments included in codegen's documents glob.

## Cache
- `typePolicies` on every type with a non-`id` key.
- `Connection` types have `keyArgs: ['filter']` (or whatever stable args matter).
- `merge` functions for paginated lists.
