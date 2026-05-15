# Input Validation

## Zod at the boundary

```ts
// === application/get-pilot-by-id.schema.ts ===
import { z } from 'zod';

export const GetPilotByIdInputSchema = z.object({
  pilotId: z.string().regex(/^P[0-9]{6,9}$/, 'Invalid pilotId format'),
});

export type GetPilotByIdInput = z.infer<typeof GetPilotByIdInputSchema>;
```

```ts
// === handler ===
const parsed = GetPilotByIdInputSchema.parse(event.arguments);
```

## Hard rules

| Rule | Severity | Source |
|------|----------|--------|
| Schema validation library missing for complex shapes | MAJOR | BE rule 49 |
| Numeric/string bounds not validated | MAJOR | BE checklist H |
| Required field missing → 500 instead of 400 | MAJOR | BE checklist H |
| Dynamic query construction from user input | CRITICAL | BE checklist H |
| AppSync `event.arguments` not null-checked | MAJOR | BE checklist H |
| Validation in handler AND use case (duplicated) | MAJOR | BE rule 51 |

## Library choice
- Prefer Zod for new code — composable schemas, strong inference.
- Match the project if Joi or class-validator is already in use.
- Hand-rolled `if (!field)` is not acceptable for any non-trivial shape.

## Failure mapping
- Validation failure → throw `ValidationError` (handled by the handler-level error mapper).
- Returns:
  - AppSync resolver: GraphQL error with `extensions.code=VALIDATION`
  - SQS consumer: log + add to `batchItemFailures` (or send to poison queue if non-retryable)
  - HTTP handler: `400 Bad Request` with the validation issues redacted

## Reuse
- Schemas live alongside use cases. They are reused by tests and by codegen for downstream contracts.
- For shared shapes (e.g., pilot id format), put the schema in `shared/schemas/` (pure, no SDK).
