# TypeScript Strict Config

## tsconfig.json (required flags)

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",

    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,

    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitOverride": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,

    "isolatedModules": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,

    "baseUrl": "./src",
    "paths": { "@/*": ["./*"] }
  }
}
```

## Hard rules (mapped to UI audit C)

| Rule | Severity | Source |
|------|----------|--------|
| Any `tsc --noEmit` error in `src/` | MAJOR per error | UI rule (C) [T1] |
| Implicit or explicit `any` | MAJOR per instance | UI rule 20 [T1] |
| Unsafe casts (`as X`) without narrowing alt | MAJOR | UI checklist C |
| `@ts-ignore` / `@ts-nocheck` without justification | MAJOR | UI checklist C |
| `!` non-null assertion | MAJOR | UI checklist C |
| Missing return types on exported fns | MAJOR | UI checklist C |
| `any` for external data (API, events) | MAJOR | UI checklist C |
| Enum with `eslint-disable no-unused-vars` per member | MAJOR — replace with `as const` | UI checklist C |
| Mock data type drift vs production interface | MAJOR | UI checklist C |
| `noUncheckedIndexedAccess` missing | MAJOR (config) | UI checklist C |
| `exactOptionalPropertyTypes` missing | MAJOR (config) | UI checklist C |
| `tsc --noEmit` not a CI gate / suppressed with `\|\| true` | MAJOR | UI rule 10 [T8] |

## Patterns

```ts
// Replace `any` from API responses with `unknown` + narrowing
type ApiResponse = unknown;

function isPilot(x: unknown): x is Pilot {
  return typeof x === 'object' && x !== null && 'pilotId' in x && typeof (x as Pilot).pilotId === 'string';
}

if (isPilot(response)) {
  // narrowed to Pilot
}
```

```ts
// Replace enum with const + type alias
export const LEAVE_STATUS = { OPEN: 'OPEN', APPROVED: 'APPROVED', REJECTED: 'REJECTED' } as const;
export type LeaveStatus = typeof LEAVE_STATUS[keyof typeof LEAVE_STATUS];
```

```ts
// Replace `!` with optional chaining + null check
const name = user?.profile?.name ?? 'Unknown';
```

```ts
// noUncheckedIndexedAccess — array access returns T | undefined
const first = items[0];
if (first !== undefined) {
  // T narrowed
}
```
