# TypeScript Config & Language Rules

## tsconfig.json (required flags)

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictPropertyInitialization": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitOverride": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "declaration": false,
    "sourceMap": true
  }
}
```

## Mandatory rules (mapped to BE audit)

| Rule | Severity | Source |
|------|----------|--------|
| `any` in production code | MAJOR per instance | BE rule 11 |
| `catch (error)` unnarrowed | MAJOR | BE rule 48 |
| `@ts-ignore` / `@ts-nocheck` without justification | MAJOR | BE checklist B |
| `!` non-null assertion | MAJOR | BE checklist B |
| Function with > 5 positional parameters | MAJOR | BE rule 59 |
| Missing return type annotations | MAJOR | BE checklist B |
| Lambda event typed as `any` | MAJOR per handler | BE rule 47 |
| `noUnusedParameters: false` | MAJOR | BE rule 42 |
| `strictPropertyInitialization: false` | MAJOR | BE rule 60 |

## Catch blocks
```ts
try {
  await doWork();
} catch (error: unknown) {
  if (error instanceof ValidationError) {
    return badRequest(error.message);
  }
  if (error instanceof Error) {
    logger.error('unexpected', { err: error });
    throw error;
  }
  throw new Error('Unknown error');
}
```

## Enums
Prefer `as const` + type alias over enums:

```ts
export const LEAVE_STATUS = {
  OPEN:     'OPEN',
  APPROVED: 'APPROVED',
  REJECTED: 'REJECTED',
} as const;

export type LeaveStatus = typeof LEAVE_STATUS[keyof typeof LEAVE_STATUS];
```

## Lambda event types
Use `aws-lambda` package types:

```ts
import type { APIGatewayProxyEvent, APIGatewayProxyResult, AppSyncResolverEvent, SQSEvent, SQSBatchResponse } from 'aws-lambda';
```

Never hand-roll the event shape. Never type the event as `any`.
