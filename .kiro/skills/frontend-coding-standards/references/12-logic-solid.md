# Logic & SOLID

## Hard rules (mapped to UI audit M)

| Rule | Severity | Source |
|------|----------|--------|
| Component does fetch + transform + render | MAJOR — split | UI checklist M |
| Business logic inside JSX render path or styled component | MAJOR | UI checklist M |
| Hook managing data fetching AND UI-specific state (modal visibility, form open/close) | MAJOR — split | UI checklist M |
| Hardcoded conditional that requires editing fn to add a new case | MAJOR — make data-driven | UI checklist M |
| Prop interface with >7 props | MAJOR — split | UI checklist M |
| Component directly importing concrete services (Apollo, fetch, localStorage) | MAJOR — inject | UI checklist M |
| Conditional nesting >2 levels | MAJOR | UI checklist M |
| >1 boolean prop on a component | MAJOR — replace with enum/union | UI checklist M |
| Unchecked array index access / optional chaining omitted | MAJOR | UI checklist M |

## Single Responsibility

```tsx
// === Smart container ===
function CrewRestPanelContainer() {
  const { data, loading, error } = useGetRestStatusQuery();
  const vm = useMemo(() => data ? mapToVm(data) : null, [data]);
  if (loading) return <CrewRestSkeleton />;
  if (error)   return <ErrorBanner error={error} />;
  if (!vm)     return <EmptyState />;
  return <CrewRestPanelView vm={vm} />;
}

// === Dumb view ===
function CrewRestPanelView({ vm }: { vm: CrewRestVm }) {
  return (
    <Stack>
      <Heading>{vm.title}</Heading>
      <RestList items={vm.items} />
    </Stack>
  );
}
```

## Replace boolean explosion with a union

```tsx
// ❌ multiple booleans
type Props = { isLoading: boolean; isError: boolean; isEmpty: boolean; isCompact: boolean };

// ✅ one variant
type Status = 'loading' | 'error' | 'empty' | 'data';
type Props  = { status: Status; compact?: boolean };
```

## Conditional nesting <= 2

```tsx
// ❌ 3 levels
if (user) {
  if (user.roles.includes('crew-supervisor')) {
    if (pilot.base === user.base) {
      return <EditableRestRow pilot={pilot} />;
    }
  }
}

// ✅ early returns
if (!user) return null;
if (!user.roles.includes('crew-supervisor')) return <ReadonlyRestRow pilot={pilot} />;
if (pilot.base !== user.base) return <ReadonlyRestRow pilot={pilot} />;
return <EditableRestRow pilot={pilot} />;
```

## Data-driven over hardcoded conditionals

```tsx
// ❌
function statusLabel(s: Status) {
  if (s === 'OPEN')     return 'Open';
  if (s === 'APPROVED') return 'Approved';
  if (s === 'REJECTED') return 'Rejected';
  return '';
}

// ✅
const STATUS_LABEL: Record<Status, string> = {
  OPEN: 'Open', APPROVED: 'Approved', REJECTED: 'Rejected',
};
const statusLabel = (s: Status) => STATUS_LABEL[s];
```

## Inject services, don't import concretes

```tsx
// ❌ component reaches into Apollo directly
import { client } from '@/lib/apollo';
function PilotRow({ id }: { id: string }) {
  const onClick = async () => {
    await client.mutate({ mutation: UpdatePilotDocument, variables: { id } });
  };
}

// ✅ via a hook
function usePilotActions() {
  const [updatePilot] = useUpdatePilotMutation();
  return { updatePilot };
}
```

## Indexed access (with `noUncheckedIndexedAccess`)

```ts
const first = items[0];          // first: T | undefined
if (first !== undefined) {
  use(first);                    // first: T
}
```
