# Verdict Format & AIQ Threading

## Verdict structure

```ts
type GuardrailVerdict = {
  passed: boolean;
  verdict: 'pass' | 'warn' | 'fail';
  checks_run: string[];
  violations: Array<{
    rule: string;
    severity: 'info' | 'warn' | 'error' | 'critical';
    detail: string;
    location?: string;
  }>;
  redactions_applied: number;
  injection_blocks: number;
  schema_validation_errors: number;
};
```

## Resolution rules

```
verdict = 'fail'  if any violation has severity in {critical, error}
verdict = 'warn'  if any violation has severity = warn AND no critical/error
verdict = 'pass'  otherwise
passed  = (verdict !== 'fail')
```

## Threading into AIQ

Every guardrail run contributes to the AIQ record:

```json
{
  "guardrails": {
    "input_pass": true,
    "output_pass": true,
    "pii_redactions": 3,
    "injection_blocks": 0,
    "schema_validation_errors": 0
  }
}
```

Threading into the handoff envelope:

```json
{
  "guardrails": {
    "input_check":  { "passed": true,  "checks_run": [...], "violations": [] },
    "output_check": { "passed": true,  "checks_run": [...], "violations": [] },
    "pii_redaction":     { "passed": true, "checks_run": [...] },
    "injection_defense": { "passed": true, "checks_run": [...] },
    "idempotency_key": "<sha256>",
    "verdict": "pass"
  }
}
```

## Downstream behaviour
- Downstream agents read the upstream verdict from the envelope.
- A `fail` verdict on the envelope itself fails handoff schema validation upstream — downstream never sees it.
- A `warn` verdict propagates: downstream records it, may proceed, but the sprint scorecard counts it.

## Reviewer side
The Code Reviewer agent treats `warn` verdicts as part of the review surface — surfaces the warnings in `REVIEW-summary.md` if not addressed by the time review runs.
