# Error Handling

## Typed error hierarchy

```ts
// shared/errors.ts
export abstract class DomainError extends Error {
  abstract readonly code: string;
  constructor(message: string, public readonly cause?: unknown) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class ValidationError extends DomainError      { readonly code = 'VALIDATION'; }
export class NotFoundError extends DomainError        { readonly code = 'NOT_FOUND'; }
export class ConflictError extends DomainError        { readonly code = 'CONFLICT'; }
export class UnauthorizedError extends DomainError    { readonly code = 'UNAUTHORIZED'; }
export class ForbiddenError extends DomainError       { readonly code = 'FORBIDDEN'; }
export class TransientError extends DomainError       { readonly code = 'TRANSIENT'; }
export class IntegrationError extends DomainError     { readonly code = 'INTEGRATION'; }
```

## Hard rules

| Rule | Severity | Source |
|------|----------|--------|
| Empty `catch {}` block | MAJOR per instance | BE rule 19 |
| Raw `throw new Error(...)` where typed error applies | MAJOR | BE checklist F |
| Error detail in API response (stack trace, internal path) | CRITICAL | BE rule 23 |
| Missing retry on transient AWS errors | MAJOR per adapter | BE rule 24 |
| `Promise.all` where partial failure must be handled | MAJOR | BE rule 28 |
| Typed error caught and returned as success response | MAJOR | BE rule 57 |
| No error masking — typed errors must not become success | MAJOR | BE checklist F |

## Translation at adapters
- DynamoDB `ConditionalCheckFailedException` → `ConflictError`.
- DynamoDB `ProvisionedThroughputExceededException` → `TransientError` (retry).
- AppSync timeout → `TransientError`.
- Validation failure (Zod) → `ValidationError`.
- Authorizer failure → `UnauthorizedError` / `ForbiddenError`.

## Translation at handlers
For AppSync resolvers:
```ts
try {
  return await useCase.execute(parsed);
} catch (err) {
  if (err instanceof NotFoundError)     throw new GraphQLError('Not Found',     { extensions: { code: 'NOT_FOUND' } });
  if (err instanceof ValidationError)   throw new GraphQLError('Invalid Input', { extensions: { code: 'VALIDATION' } });
  if (err instanceof UnauthorizedError) throw new GraphQLError('Unauthorized',  { extensions: { code: 'UNAUTHORIZED' } });
  // unexpected → log full err, throw a generic message
  logger.error('unexpected', { err });
  throw new GraphQLError('Internal error', { extensions: { code: 'INTERNAL' } });
}
```

## Async / await rules
- Every `async` function has a `try/catch` or a `.catch()` (BE checklist F).
- No unhandled-promise patterns (`void doWork()` at module scope).
- `Promise.allSettled` for fan-out where partial failure must be handled (BE rule 28).
- Retry with exponential backoff on transient errors (handled by SDK retry strategy + adapter wrappers where AWS retry isn't enough).
