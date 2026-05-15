# Frontend Architecture

## Feature-based folder layout

```
src/
├── features/
│   ├── crew-rest-violation/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/        # GraphQL operations
│   │   ├── types/
│   │   ├── __tests__/
│   │   ├── index.ts         # barrel — exports page component(s) only
│   │   └── README.md        # only when non-obvious operational concerns
│   ├── leave-management/
│   └── ...
├── shared/                   # cross-feature only (≥2 features use it)
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── types/
├── lib/                      # infrastructure only — Apollo client, logger, test utils
│   ├── apollo/
│   ├── logger/
│   └── test-utils/
├── routes/
│   ├── index.tsx
│   └── guards/
├── theme/                    # design tokens + theme provider
└── __generated__/            # codegen output (graphql.ts, etc.)
```

## Hard rules (mapped to UI audit D)

| Rule | Severity | Source |
|------|----------|--------|
| Cross-feature import (feature A → feature B internals) | CRITICAL (boundary) | UI checklist D |
| `/shared` contains single-feature code | MAJOR | UI checklist D |
| `/lib` contains business logic | MAJOR | UI checklist D |
| Feature barrel exports anything other than page components | MAJOR | UI checklist D |
| Circular dependencies | MAJOR | UI rule (D) [T13] |
| Module-level mutable singleton (`const x = new SomeClass()`) | MAJOR | UI rule 12 [T15] |
| Each feature folder missing one of: `components/` `hooks/` `types/` `services/` `__tests__/` | MAJOR | UI checklist D |
| Rogue `utils/` at repo root | MAJOR | UI checklist D |

## Naming
- Components: `PascalCase.tsx` (`CrewRestPanel.tsx`).
- Hooks: `useX.ts` (`useCrewRest.ts`).
- Utils / pure modules: `kebab-case.ts` (`parse-local-date.ts`).
- Types-only files: `<concern>.types.ts`.
- Styled components: `<Component>.styled.tsx` if CSS-in-JS is the project default.

## Cross-feature dependencies
- Allowed only via `/shared` or `/lib`.
- Direct `import { X } from 'features/<other>/...` from outside the feature is a boundary violation.
- ESLint rule `import/no-internal-modules` enforces this.

## Module-level singletons
- Avoid: `const apolloClient = new ApolloClient(...)` at module scope inside `src/`.
- Place in `lib/apollo/` with explicit `getClient()` factory; consumed via context.
- Detection: T15 (`grep -rn "^const .* = new " src/`).

## Circular dependencies
- `madge --circular src/` runs in CI.
- Any reported cycle is MAJOR; report the full import chain.
