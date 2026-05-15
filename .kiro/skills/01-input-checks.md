# Input Sanity Checks

## What runs at every agent's Phase 0

```ts
type GuardrailVerdict = {
  passed: boolean;
  checks_run: string[];
  violations: Array<{
    rule: string;
    severity: 'info' | 'warn' | 'error' | 'critical';
    detail: string;
  }>;
};

async function runInputCheck(input: AgentInput): Promise<GuardrailVerdict> {
  const violations: GuardrailVerdict['violations'] = [];
  const checks_run: string[] = [];

  // 1. Size limit
  checks_run.push('size-limit');
  if (input.totalBytes() > 256 * 1024) {
    violations.push({ rule: 'size-limit', severity: 'error', detail: `> 256KB: ${input.totalBytes()}` });
  }

  // 2. Encoding (must be UTF-8)
  checks_run.push('encoding');
  if (!input.isUtf8()) {
    violations.push({ rule: 'encoding', severity: 'error', detail: 'non-UTF8 bytes detected' });
  }

  // 3. Schema (handoff envelope, when present)
  checks_run.push('handoff-schema');
  if (input.handoffPath && !validateAgainstSchema(input.handoffPath, HANDOFF_ENVELOPE_SCHEMA)) {
    violations.push({ rule: 'handoff-schema', severity: 'critical', detail: 'envelope failed schema' });
  }

  // 4. Allow-list (Confluence/Jira/Figma resource ids)
  checks_run.push('allow-list');
  for (const ref of input.externalRefs()) {
    if (!ALLOW_LIST.includes(ref)) {
      violations.push({ rule: 'allow-list', severity: 'critical', detail: `out-of-scope: ${ref}` });
    }
  }

  // 5. Injection markers (see 02-injection-defense.md)
  checks_run.push('injection-markers');
  const injection = detectInjectionMarkers(input.text());
  if (injection.matched) {
    violations.push({ rule: 'injection', severity: 'warn', detail: injection.summary });
  }

  // 6. PII present in input — flag, redact downstream
  checks_run.push('pii-input-scan');
  const pii = detectPII(input.text());
  if (pii.classes.includes('high')) {
    violations.push({ rule: 'pii-high', severity: 'error', detail: 'High-class PII present in input' });
  }

  return {
    passed: violations.every((v) => v.severity !== 'critical' && v.severity !== 'error'),
    checks_run,
    violations,
  };
}
```

## Verdict thresholds
- `passed: false` (any error or critical) → HALT the agent turn. Surface to user.
- Warnings only → proceed; record warnings on the AIQ line.
- Critical → also emit a decision-audit record so the refusal is queryable.
