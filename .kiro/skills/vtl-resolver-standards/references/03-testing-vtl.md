# Testing VTL on the AppSync Simulator

## Tooling
- `amplify-appsync-simulator` for local execution.
- Velocity-template-language interpreter compatible with AppSync's VTL flavor.
- `aws-appsync-mock` (or hand-rolled fixtures) for `$ctx`.

## Test layout
```
tests/vtl/
├── fixtures/
│   ├── ctx-pilot-by-id.json
│   ├── ctx-pilot-by-id-not-found.json
│   └── ...
├── golden/
│   ├── Query.pilotById.expected-req.json
│   ├── Query.pilotById.expected-res.json
│   └── ...
├── runner.ts
└── Query.pilotById.test.ts
```

## Golden test pattern

```ts
import { test, expect } from 'vitest';
import { evaluateVtl } from './runner';
import ctx from './fixtures/ctx-pilot-by-id.json';
import expectedReq from './golden/Query.pilotById.expected-req.json';

test('Query.pilotById request template', async () => {
  const out = await evaluateVtl('schema/vtl/Query.pilotById.req.vtl', ctx);
  expect(out).toEqual(expectedReq);
});

test('Query.pilotById response template (happy path)', async () => {
  const ddbResult = { pilotId: 'P123', name: 'Test Pilot', base: 'DAL' };
  const out = await evaluateVtl('schema/vtl/Query.pilotById.res.vtl', { ...ctx, result: ddbResult });
  expect(out).toEqual(ddbResult);
});

test('Query.pilotById response template (not found)', async () => {
  await expect(
    evaluateVtl('schema/vtl/Query.pilotById.res.vtl', { ...ctx, result: null })
  ).rejects.toMatchObject({ type: 'NOT_FOUND' });
});
```

## Required test cases per resolver

| # | Case | Verifies |
|---|------|----------|
| 1 | Happy-path req | request template renders DynamoDB request correctly |
| 2 | Happy-path res | response template returns expected shape |
| 3 | Error branch | `$util.error(...)` fires with the right type |
| 4 | Not-found / null result | error type `NOT_FOUND` |
| 5 | Auth condition (where applicable) | `condition` / `expressionValues` shape |
| 6 | Identity propagation | `updatedBy = $ctx.identity.username` propagated |

## Coverage gate
Per pilot: every VTL template has at least cases 1–4. Coverage measured by file count, not line count
(Velocity is template language; line coverage is misleading).
