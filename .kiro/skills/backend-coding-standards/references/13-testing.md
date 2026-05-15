# Testing Standards

## Layer targets

| Layer | Target |
|-------|--------|
| `domain/` | ≥ 95% statements |
| `application/` (use cases) | ≥ 90% |
| `adapters/outbound/` | ≥ 80% |
| `adapters/inbound/` (handlers) | ≥ 70% |
| `shared/` utilities | ≥ 95% |

## Hard rules (mapped to BE audit)

| Rule | Severity | Source |
|------|----------|--------|
| Coverage < 80% on domain/application | MAJOR | BE rule (L) |
| Tests use real AWS SDK (not mocked) in unit tests | MAJOR | BE checklist L |
| `(obj as any).privateMethod` to access internals | MAJOR | BE rule 37 |
| `jest.doMock` inside test body | MAJOR | BE rule 38 |
| `jest.clearAllMocks()` not in `beforeEach` | MAJOR | BE rule 50 |
| Coverage-padding test (only `toHaveProperty`, named `coverage-boost`) | MAJOR per file | BE rule 36 |
| Mock data type mismatched with production interface | MAJOR | BE checklist L |
| Snapshot test on logic-heavy component | MAJOR | BE checklist L |
| `services/**` excluded from coverage | MAJOR | BE checklist J |

## Mock strategy

```ts
// Unit test for a use case — mock the port
const repo: jest.Mocked<PilotRepositoryPort> = {
  findById: jest.fn(),
  listByBase: jest.fn(),
};

const useCase = new GetPilotByIdUseCase(repo);

test('returns pilot when found', async () => {
  repo.findById.mockResolvedValue({ pilotId: 'P123', name: 'Test', base: 'DAL' });
  await expect(useCase.execute({ pilotId: 'P123' })).resolves.toEqual({ pilotId: 'P123', name: 'Test', base: 'DAL' });
});

test('throws NotFoundError when missing', async () => {
  repo.findById.mockResolvedValue(null);
  await expect(useCase.execute({ pilotId: 'P999' })).rejects.toThrow(NotFoundError);
});
```

```ts
// Adapter test — aws-sdk-client-mock for SDK v3
import { mockClient } from 'aws-sdk-client-mock';
import { DynamoDBDocumentClient, GetCommand } from '@aws-sdk/lib-dynamodb';

const ddbMock = mockClient(DynamoDBDocumentClient);

beforeEach(() => ddbMock.reset());

test('findById maps DDB shape to domain', async () => {
  ddbMock.on(GetCommand).resolves({ Item: { pilotId: 'P123', name: 'Test', base: 'DAL' } });
  const repo = new DdbPilotRepository();
  await expect(repo.findById('P123')).resolves.toEqual({ pilotId: 'P123', name: 'Test', base: 'DAL' });
});
```

## Property-based tests (where it pays off)
- Use `fast-check` for pure domain functions with non-trivial input space.
- Required for date arithmetic, financial calculations, schedule windows.

## Edge cases (every use case)
- Empty / null inputs
- Boundary values
- Error paths (downstream failures)
- Concurrent invocations (idempotency-replay)

## Test data factories
- Pure deterministic functions in `tests/factories/`.
- Seeded RNG only; no `Math.random()` directly in a test.
- Factories produce types that match production interfaces (no drift).
