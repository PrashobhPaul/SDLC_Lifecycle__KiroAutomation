# Data Fetching & Caching

## Hard rules (mapped to UI audit I)

| Rule | Severity | Source |
|------|----------|--------|
| Query without explicit `fetchPolicy` + comment | MAJOR per callsite | UI checklist I |
| Mutation without optimistic response or refetch fallback | MAJOR | UI checklist I |
| Mutation without defined cache update strategy | MAJOR | UI checklist I |
| `useQuery` / `useMutation` / `useLazyQuery` without `loading` handling | MAJOR per callsite | UI rule 28 |
| `useQuery` / `useMutation` / `useLazyQuery` without `error` handling | MAJOR per callsite | UI rule 27 |
| Retry on 4xx (auth failure) | MAJOR | UI checklist I |
| Waterfall request (B depends on A's result, executed serially) | MAJOR per occurrence | UI checklist I |
| Cancellable request without `AbortController` | MAJOR | UI checklist I |
| Update state on unmounted component | MAJOR | UI checklist I |
| Custom cache layer without rationale | MAJOR | UI checklist I |

## fetchPolicy + comment

```tsx
// network-only: balance must reflect latest data after a write
const { data, loading, error } = usePilotBalanceQuery({
  variables: { pilotId },
  fetchPolicy: 'network-only',
});
```

## Optimistic mutation + cache update

```tsx
const [updatePilot] = useUpdatePilotMutation({
  optimisticResponse: ({ input }) => ({
    __typename: 'Mutation' as const,
    updatePilot: { __typename: 'Pilot', ...currentPilot, ...input },
  }),
  update: (cache, { data }) => {
    if (!data?.updatePilot) return;
    cache.writeQuery({
      query: GetPilotByIdDocument,
      variables: { pilotId: data.updatePilot.pilotId },
      data: { pilotById: data.updatePilot },
    });
  },
});
```

## Always handle loading + error

```tsx
const { data, loading, error } = useGetPilotByIdQuery({ variables: { pilotId } });

if (loading) return <SkeletonCard />;
if (error)   return <ErrorBanner error={error} retry={() => refetch()} />;
if (!data?.pilotById) return <EmptyState />;

return <PilotCard pilot={data.pilotById} />;
```

## Retry policy
- Retry on 5xx and network errors with exponential backoff.
- Never retry on 4xx (especially 401/403 — retry just amplifies an auth failure).
- Apollo `RetryLink` configured at the link layer with `attempts: { max: 3, retryIf: shouldRetry }`.

## Cancellation

```tsx
useEffect(() => {
  const ctrl = new AbortController();
  fetch(`/api/search?q=${q}`, { signal: ctrl.signal })
    .then(r => r.json())
    .then(setResults)
    .catch(err => { if (err.name !== 'AbortError') logger.error(err); });
  return () => ctrl.abort();
}, [q]);
```

## No waterfalls

```tsx
// ❌ serial
const { data: pilot }    = useGetPilotByIdQuery({ variables: { pilotId } });
const { data: schedule } = useGetScheduleQuery({ variables: { pilotId } });

// ✅ parallel via combined query OR independent hooks; the cache layer dedupes
const [pilotResult, scheduleResult] = await Promise.all([
  client.query({ query: GetPilotByIdDocument, variables: { pilotId } }),
  client.query({ query: GetScheduleDocument,  variables: { pilotId } }),
]);
```
