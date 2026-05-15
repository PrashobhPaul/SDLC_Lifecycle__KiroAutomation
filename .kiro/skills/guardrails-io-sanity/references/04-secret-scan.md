# Secret & Credential Scanning at the Output Boundary

## Why this is mandatory
Agents generate code, Terraform, env-var fallbacks, and example payloads. Any of these can leak a credential
captured upstream. The output boundary scan is the last line before an artifact lands on disk.

## Patterns

```ts
const SECRET_PATTERNS: Array<{ name: string; rx: RegExp; severity: 'critical' | 'error' }> = [
  { name: 'aws-access-key', rx: /\b(AKIA|ASIA)[0-9A-Z]{16}\b/, severity: 'critical' },
  { name: 'aws-secret-key', rx: /\b[a-zA-Z0-9/+=]{40}\b/, severity: 'critical' },
  { name: 'aws-session-token', rx: /\baws_session_token\s*=\s*['"][A-Za-z0-9/+=]{200,}['"]/i, severity: 'critical' },
  { name: 'aws-account-id', rx: /\b\d{12}\b/, severity: 'error' },
  { name: 'github-pat', rx: /\bghp_[A-Za-z0-9]{36}\b/, severity: 'critical' },
  { name: 'gitlab-pat', rx: /\bglpat-[A-Za-z0-9_\-]{20}\b/, severity: 'critical' },
  { name: 'jwt', rx: /\beyJ[A-Za-z0-9_\-]+\.eyJ[A-Za-z0-9_\-]+\.[A-Za-z0-9_\-]+\b/, severity: 'critical' },
  { name: 'private-key', rx: /-----BEGIN (RSA |EC |OPENSSH |DSA )?PRIVATE KEY-----/, severity: 'critical' },
  { name: 'slack-token', rx: /\bxox[baprs]-[A-Za-z0-9-]{10,}\b/, severity: 'critical' },
  { name: 'generic-secret-assignment', rx: /\b(password|passwd|secret|api_key|apikey|token|credential|access_key)\s*[:=]\s*['"][^'"\n]{8,}['"]/i, severity: 'error' },
  { name: 'pem-fragment', rx: /-----BEGIN CERTIFICATE-----/, severity: 'error' },
];
```

## Behaviour on match
- `severity=critical` → HALT the write. Artifact is NOT persisted. Surface to user with the rule and a redacted location (line numbers, no value).
- `severity=error` → HALT and surface, allowing user to confirm the value is a placeholder (e.g., a dummy AWS account id in an example).
- Every match emits a decision-audit record with `category=guardrail`.

## False-positive handling
- A 12-digit number is often a phone or order id, not an AWS account id. The detector requires the surrounding context to look like an AWS Terraform/IAM string.
- Generic secret assignment is suppressed for `< 8 chars` (likely a placeholder).
- Tests / fixtures may contain dummy creds; the detector inspects the file path. Inside `tests/`, `fixtures/`, or files matching `*.example.*`, severity is downgraded to `warn`.

## Tool integration
- Pre-commit: `gitleaks detect --redact --staged`.
- CI: `gitleaks detect --redact` on the full repo.
- Output boundary (this skill): same patterns, applied in-memory before write.

## Where to find the master list
- Master patterns versioned at `tools/secret-patterns.json`.
- Updates require AI-COE Lead review (signed commit).
