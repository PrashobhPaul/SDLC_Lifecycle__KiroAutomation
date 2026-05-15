# Data Processing (Frontend)

## Hard rules (mapped to UI audit J)

| Rule | Severity | Source |
|------|----------|--------|
| Sort/filter/map on large dataset inside render or without `useMemo` | MAJOR per instance | UI checklist J |
| Mapper with side effects (API calls, state mutations, logging) | MAJOR | UI checklist J |
| Transformation logic inside a component's render | MAJOR | UI checklist J |
| Date/number/currency formatter inline in JSX (not in `utils/`) | MAJOR | UI checklist J |
| `new Date(dateString)` on a date-only string | MAJOR | UI checklist J |
| Float arithmetic without `Math.floor` / `Math.round` / `toFixed` | MAJOR | UI checklist J |

## parseLocalDate pattern

```ts
// shared/utils/parse-local-date.ts
export function parseLocalDate(input: string): Date {
  // `new Date('2024-01-15')` parses as UTC midnight — rolls back to 2024-01-14 in negative-offset locales.
  const m = /^(\d{4})-(\d{2})-(\d{2})$/.exec(input);
  if (!m) throw new Error(`Invalid date-only string: ${input}`);
  return new Date(Number(m[1]), Number(m[2]) - 1, Number(m[3]));
}
```

## Pure mappers

```ts
// shared/mappers/pilot.mapper.ts
import type { GetPilotByIdQuery } from '@/__generated__/graphql';
import type { PilotVm } from './pilot.types';

export function mapPilotToVm(p: NonNullable<GetPilotByIdQuery['pilotById']>): PilotVm {
  return {
    id:       p.pilotId,
    name:     p.name,
    base:     p.base,
    initials: p.name.split(' ').map(s => s[0]).join('').slice(0, 2).toUpperCase(),
  };
}
```

No React hooks, no API calls, no logger. Easy to test.

## Memoise heavy transforms

```tsx
const sortedItems = useMemo(
  () => items.toSorted((a, b) => a.priority - b.priority),
  [items]
);
```

## Centralised formatters

```ts
// shared/utils/format.ts
export const fmt = {
  date: (d: Date) => new Intl.DateTimeFormat('en-US', { dateStyle: 'medium' }).format(d),
  time: (d: Date) => new Intl.DateTimeFormat('en-US', { timeStyle: 'short'  }).format(d),
  duration: (mins: number) => `${Math.floor(mins / 60)}h ${mins % 60}m`,
  currency: (n: number) => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(n),
};
```
