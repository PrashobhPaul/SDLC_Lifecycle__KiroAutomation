# State Management

## Hard rules (mapped to UI audit F)

| Rule | Severity | Source |
|------|----------|--------|
| Global state only used by one component | MAJOR | UI checklist F |
| Prop drilling >1 level | MAJOR | UI rule 22 |
| Missing/stale dep in `useEffect` / `useCallback` deps array | MAJOR per instance | UI checklist F |
| Missing `useMemo` / `useCallback` / `React.memo` on hot render path | MAJOR | UI checklist F |
| Derived state stored in `useState` instead of computed | MAJOR | UI checklist F |
| >2 related `useState` calls in one component | MAJOR — refactor to `useReducer` | UI rule 23 |
| `finally` block that skips cleanup when request stale | MAJOR | UI checklist F |
| Array index as key on reorderable / paginated list | MAJOR per instance | UI checklist F |
| `useState` mirroring data already in fetch cache | MAJOR | UI checklist F |

## Lowest-scope rule

```tsx
// State that's only consumed by one component lives there.
function PilotCard({ pilot }: { pilot: Pilot }) {
  const [expanded, setExpanded] = useState(false);
  return <Card onClick={() => setExpanded(v => !v)}>...</Card>;
}
```

Lifting state higher than the closest common ancestor is the most common cause of unnecessary re-renders.

## useReducer threshold

```tsx
// >2 related useState — refactor
type State = { isLoading: boolean; data: Pilot | null; error: Error | null };
type Action =
  | { type: 'fetch-start' }
  | { type: 'fetch-success'; data: Pilot }
  | { type: 'fetch-error'; error: Error };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'fetch-start':   return { ...state, isLoading: true, error: null };
    case 'fetch-success': return { isLoading: false, data: action.data, error: null };
    case 'fetch-error':   return { isLoading: false, data: null, error: action.error };
  }
}

const [state, dispatch] = useReducer(reducer, { isLoading: false, data: null, error: null });
```

## Stable list keys

```tsx
// ❌ index key on reorderable list
items.map((item, i) => <Row key={i} item={item} />);

// ✅ stable id
items.map(item => <Row key={item.id} item={item} />);
```

## Effect dep arrays

ESLint rule `react-hooks/exhaustive-deps` set to `"error"`, not `"warn"`. Stale closures are real bugs.

## Don't mirror cache

```tsx
// ❌ mirroring Apollo cache
const { data } = useQuery(GET_PILOT, { variables: { id } });
const [pilot, setPilot] = useState(data?.pilot);
useEffect(() => setPilot(data?.pilot), [data]);

// ✅ read directly
const { data } = useQuery(GET_PILOT, { variables: { id } });
const pilot = data?.pilot;
```

## Stale-request cleanup

```tsx
useEffect(() => {
  let cancelled = false;
  fetchData().then(d => { if (!cancelled) setData(d); });
  return () => { cancelled = true; };
}, [id]);
```
