# Security

## Secrets & credentials

| Rule | Severity | Source |
|------|----------|--------|
| Hardcoded secret/credential in source or Terraform | CRITICAL | BE rule 12 |
| Secret as plain Lambda env var | CRITICAL | BE checklist E |
| `dotenv` in production Lambda source | MAJOR | BE rule 55 |
| Hardcoded fallback (`\|\| 'us-east-1'`, `\|\| 'local'`) | MAJOR | BE rule 30 |

## Secrets management
- Secrets via AWS Secrets Manager (with rotation) or SSM Parameter Store (encrypted strings).
- Resolved at cold start; cached in memory for the execution context.
- Never logged.

```ts
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: process.env.AWS_REGION });
let _cached: Map<string, string> | undefined;

export async function getSecret(name: string): Promise<string> {
  if (!_cached) _cached = new Map();
  const cached = _cached.get(name);
  if (cached) return cached;
  const out = await client.send(new GetSecretValueCommand({ SecretId: name }));
  if (!out.SecretString) throw new Error(`Secret ${name} has no SecretString`);
  _cached.set(name, out.SecretString);
  return out.SecretString;
}
```

## IAM least-privilege
- Per Terraform standards (`terraform-aws-standards/references/05-iam-least-privilege.md`).
- IAM action ↔ SDK call cross-reference (BE rule 61).

## TLS
- Outbound HTTPS only.
- `rejectUnauthorized: false` is forbidden (CRITICAL).
- Use the AWS-provided trust store; do not bundle CA certs.

## CORS
- AppSync handles CORS at the API layer.
- For any non-AppSync HTTP endpoint, allow-list explicit origins; never `*` in prod.

## Input sanitisation
- All external inputs validated and typed before use (see `12-input-validation.md`).
- No dynamic SQL/NoSQL built from user strings.
- AppSync `event.arguments` null-checked before use.

## PII
- Logs, traces, and metrics never include high-class PII (SSN, passport, full DOB).
- Medium PII (names, emails, employee IDs) redacted via the `redactor` utility before logging.

## Vulnerability scanning
- `npm audit` in CI. CRITICAL/HIGH = block (BE rule 3 / E).
- Renovate or Dependabot enabled (BE rule 67).
- License audit: no GPL/AGPL in commercial product (BE checklist J).

## Build artifacts
- `node_modules`, `dist`, `.terraform`, `.venv`, `*.backup`, `debug.json`, `junit.xml` in `.gitignore` (BE checklist J).
- Build outputs never committed.
