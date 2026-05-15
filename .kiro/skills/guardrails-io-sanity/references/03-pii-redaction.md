# PII Redaction

## PII classes (Phase 1 scope)

| Class | Examples | Action |
|-------|---------|--------|
| high | SSN, passport number, full DOB, full home address | Redact at boundary; refuse if originating outside SWA scope |
| medium | Full employee name, employee ID, full email | Redact in logs; allow in code with audit |
| low | Initials, partial date (YYYY-MM), domain only | Allow with note |
| none | Synthetic / placeholder values | Allow |

## Detection

```ts
const PATTERNS: Array<{ class: 'high' | 'medium' | 'low'; rx: RegExp }> = [
  { class: 'high', rx: /\b\d{3}-\d{2}-\d{4}\b/ },                                  // SSN
  { class: 'high', rx: /\b[A-Z]{1,2}\d{6,9}\b/ },                                   // Passport
  { class: 'high', rx: /\b(19|20)\d{2}-\d{2}-\d{2}\b.*\b(dob|birth)\b/i },         // DOB context
  { class: 'medium', rx: /\b(emp_id|employee[_\s]?id)[^a-z0-9]([a-z0-9]{6,})/i },  // Emp id
  { class: 'medium', rx: /\b[A-Za-z0-9._%+-]+@swa\.[a-z]{2,}\b/i },                // SWA email
  { class: 'medium', rx: /\b(pilot|crew|fa)[_\s]?id[^a-z0-9]([a-z0-9]{6,})/i },    // Crew id
  { class: 'low', rx: /\b[A-Z]\.\s?[A-Z]\b/ },                                     // Initials
];
```

## Redaction strategy
- Replace match with `[REDACTED:<class>]` in logs and AIQ records.
- Replace match with stable hash `pii_<sha256[:8]>` in artifacts that need a stable id.
- Allow original value only inside HITL-protected handoff envelopes when essential to the feature.

## Allow-list
A PII allow-list lives in `.kiro/steering/pii-allow-list.md` (pointer) for cases where a real value
must traverse the agent (e.g., crew member id used as a join key for a generated SQL).
The allow-list is HITL-approved and time-bounded.

## Where it runs
- Input check: detect-only; record `guardrails.pii_redactions` count.
- Output check: redact. Halt the write if `class=high` is detected and the artifact is going outside the SWA scope.
- Log boundary: redact unconditionally before writing JSONL records.
