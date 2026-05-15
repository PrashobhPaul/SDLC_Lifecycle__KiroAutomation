# Output Schema Validation

## Contracts every output is checked against

| Artifact | Schema | Owner |
|----------|--------|-------|
| Handoff envelope | `.kiro/handoff/schemas/handoff-envelope.schema.json` | feature-builder, developer, test-engineer, code-reviewer |
| AIQ scorecard line | `.kiro/schemas/aiq-scorecard.schema.json` | every agent |
| Question / answer file | `.kiro/schemas/question-back.schema.json` | every agent |
| Pointer file | implicit, validated by `pointer-schema-linter` against `.kiro/pointer-schema/POINTER_SCHEMA.md` | linter |

## Validator

```ts
import Ajv from 'ajv';
import addFormats from 'ajv-formats';

const ajv = new Ajv({ allErrors: true, strict: true });
addFormats(ajv);

export function validateAgainstSchema(data: unknown, schema: object): { valid: boolean; errors: string[] } {
  const validate = ajv.compile(schema);
  const valid = validate(data);
  return {
    valid: !!valid,
    errors: (validate.errors ?? []).map((e) => `${e.instancePath} ${e.message}`),
  };
}
```

## Behaviour on failure
1. The artifact is NOT written.
2. The agent surfaces the violation list with file path, instance path, and the failing rule.
3. AIQ record records `guardrails.schema_validation_errors++`.
4. Decision-audit record written with `category=guardrail`.

## Behaviour for downstream agents
A downstream agent **refuses to start** if a referenced upstream handoff fails validation.
This protects the chain integrity (handshake fidelity ≥ 99% target).
