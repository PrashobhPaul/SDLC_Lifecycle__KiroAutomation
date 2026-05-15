# GraphQL on the Frontend

## Hard rules (mapped to UI audit V)

| Rule | Severity | Source |
|------|----------|--------|
| `useQuery`/`useMutation`/`useLazyQuery` not using generated types | MAJOR per callsite | UI checklist V |
| String template interpolation into `gql` tag | MAJOR | UI checklist V |
| Same query/mutation defined in >1 file | MAJOR | UI checklist V |
| Query fetches fields not rendered by consumer | MAJOR per field | UI checklist V |
| `useQuery` / `useMutation` without `error` handling | MAJOR per callsite | UI rule 27 |
| `useQuery` / `useMutation` without `loading` handling | MAJOR per callsite | UI rule 28 |
| `useQuery` firing before required vars ready (no `skip`) | MAJOR | UI checklist V |
| Mutation modifies data without updating Apollo cache | MAJOR | UI checklist V |
| `useSubscription` without cleanup | MAJOR | UI checklist V |
| List component firing query per item (N+1) | MAJOR | UI checklist V |
| Query without explicit `fetchPolicy` | MAJOR | UI checklist V |
| `services/**` excluded from `collectCoverageFrom` | MAJOR | UI checklist V [T16] |

## Codegen (single source of types)

```yaml
# codegen.yml
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
      avoidOptionals: { field: true }
      enumsAsTypes: true
      scalars:
        AWSDateTime: string
        AWSDate: string
        AWSJSON: 'Record<string, unknown>'
```

```bash
npm run codegen   # runs on save of any .graphql via the onFileSave hook
```

## Operation files

```graphql
# src/features/crew-rest-violation/services/get-rest-status.graphql
query GetRestStatus($pilotId: ID!) {
  pilotById(pilotId: $pilotId) {
    ...PilotProfile
    restRemainingMinutes
    nextDutyStart
  }
}

fragment PilotProfile on Pilot {
  pilotId
  name
  base
}
```

## Generated hook usage

```tsx
const { data, loading, error, refetch } = useGetRestStatusQuery({
  variables: { pilotId },
  fetchPolicy: 'cache-and-network',
  skip: !pilotId,
});

if (loading && !data) return <Skeleton />;
if (error)            return <ErrorBanner error={error} retry={() => refetch()} />;
if (!data?.pilotById) return <EmptyState />;
return <PilotRestPanel pilot={data.pilotById} />;
```

## No string-template interpolation

```ts
// ❌ Apollo cannot parse / cache fragments built like this
const query = gql`query { pilot(id: "${id}") { ${FIELDS} } }`;

// ✅ proper fragment + variable
const PILOT_FIELDS = gql`fragment PilotProfile on Pilot { pilotId name base }`;
const QUERY = gql`
  query GetPilot($id: ID!) {
    pilotById(pilotId: $id) {
      ...PilotProfile
    }
  }
  ${PILOT_FIELDS}
`;
```

## Cache update on mutation

```tsx
const [updatePilot] = useUpdatePilotMutation({
  update(cache, { data }) {
    if (!data?.updatePilot) return;
    cache.modify({
      id: cache.identify({ __typename: 'Pilot', pilotId: data.updatePilot.pilotId }),
      fields: {
        base: () => data.updatePilot.base,
        updatedAt: () => data.updatePilot.updatedAt,
      },
    });
  },
});
```

## Subscriptions cleanup

```tsx
useEffect(() => {
  const sub = client
    .subscribe({ query: OnPilotUpdatedDocument, variables: { pilotId } })
    .subscribe({ next: (msg) => onUpdate(msg.data?.pilotUpdated) });
  return () => sub.unsubscribe();
}, [pilotId]);
```

## No N+1

```tsx
// ❌ list calls a query per item
items.map(item => <PilotChip key={item.id} useGetPilotByIdQuery={...} />);

// ✅ list query with multi-id resolver
const { data } = useGetPilotsByIdsQuery({ variables: { ids } });
```
