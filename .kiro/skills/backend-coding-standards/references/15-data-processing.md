# Data Processing

## Hard rules (mapped to BE audit)

| Rule | Severity | Source |
|------|----------|--------|
| Mapper / transformer with side effects (API calls, state mutations, logging) | MAJOR per instance | BE rule 22 |
| Transformation in handler render path | MAJOR | BE checklist K |
| Date / number / currency formatters not centralised in `utils/` | MAJOR | BE checklist K |
| `new Date(dateString)` on a date-only string | MAJOR | BE checklist K |
| Float arithmetic on financial values without `Math.round` / `toFixed` / decimal lib | MAJOR | BE checklist K |
| Unbounded array accumulation across pagination pages | MAJOR | BE rule 26 |
| Large dataset sort/filter in Lambda memory (push to DB) | MAJOR | BE checklist K |
| Raw SDK shape leaking into domain | MAJOR | BE rule 70 |
| Null/undefined DB response fields not handled explicitly | MAJOR | BE checklist K |
| Static data files >100 KB inside `src/` | MAJOR | BE rule 40 |

## Mappers are pure

```ts
// === shared/mappers/pilot.mapper.ts ===
import type { Pilot } from '../../domain/pilot';

export function pilotFromDdb(item: Record<string, unknown>): Pilot {
  return {
    pilotId:   String(item.pilotId),
    name:      String(item.name),
    base:      String(item.base),
    updatedAt: String(item.updatedAt),
  };
}

export function pilotToDdb(p: Pilot): Record<string, unknown> {
  return {
    pk: `PILOT#${p.pilotId}`,
    sk: 'PROFILE',
    pilotId: p.pilotId,
    name: p.name,
    base: p.base,
    updatedAt: p.updatedAt,
  };
}
```

No SDK calls, no logger, no I/O. Easy to unit test, easy to reason about.

## Date-only parsing — `parseLocalDate` pattern

```ts
// shared/time.ts
export function parseLocalDate(input: string): Date {
  // Avoid `new Date('2024-01-15')` — it is parsed as UTC midnight, which can roll back to the previous day in negative-offset locales.
  const m = /^(\d{4})-(\d{2})-(\d{2})$/.exec(input);
  if (!m) throw new Error(`Invalid date-only string: ${input}`);
  return new Date(Number(m[1]), Number(m[2]) - 1, Number(m[3]));
}
```

## Float arithmetic on financial values

```ts
// ❌ wrong
const total = price * quantity;          // 0.1 * 0.2 → 0.020000000000000004

// ✅ correct (basic)
const total = Math.round(price * quantity * 100) / 100;

// ✅ correct (high-precision)
import Decimal from 'decimal.js';
const total = new Decimal(price).mul(quantity).toDecimalPlaces(2).toNumber();
```

## Unbounded accumulation
- Always cap accumulators with the smallest reasonable upper bound per use case.
- Stream large outputs to S3 or downstream APIs incrementally; never hold the entire dataset in memory.
