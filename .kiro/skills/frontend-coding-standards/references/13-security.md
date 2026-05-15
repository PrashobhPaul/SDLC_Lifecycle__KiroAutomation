# Frontend Security

## Hard rules (mapped to UI audit N)

| Rule | Severity | Source |
|------|----------|--------|
| `dangerouslySetInnerHTML` without DOMPurify | CRITICAL per instance | UI rule N |
| Secret / API key in `src/` or committed `.env` | CRITICAL | UI checklist N |
| UI-gated logic without server-side authz | MAJOR | UI checklist N |
| `href="javascript:"` or template-injected URL | CRITICAL | UI checklist N |
| CSV export without formula-injection prevention | MAJOR | UI checklist N |
| State-mutating request without CSRF token / `SameSite=Strict\|Lax` | MAJOR | UI checklist N |
| Sensitive data in `localStorage` / `sessionStorage` unencrypted | MAJOR | UI checklist N |
| `<iframe>` without `sandbox` | MAJOR | UI checklist N |
| Open-redirect from user-supplied URL | CRITICAL | UI checklist N |
| CSP not configured at server / CDN level | MAJOR (meta-tag CSP is bypassable) | UI checklist N |
| `npm audit` CRITICAL/HIGH in prod scope | CRITICAL | UI checklist N [T3] |

## DOMPurify wrapper

```tsx
import DOMPurify from 'dompurify';

export function SafeHtml({ html }: { html: string }) {
  const clean = useMemo(
    () => DOMPurify.sanitize(html, {
      ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'a', 'ul', 'ol', 'li'],
      ALLOWED_ATTR: ['href', 'target', 'rel'],
    }),
    [html]
  );
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

## CSV formula-injection prevention

```ts
// shared/utils/csv.ts
export function escapeCsvCell(value: unknown): string {
  const s = String(value ?? '');
  if (/^[=+\-@]/.test(s)) return `'${s}`;   // prefix to neutralise formula
  if (/["\n,]/.test(s))   return `"${s.replace(/"/g, '""')}"`;
  return s;
}
```

## URL validation for redirects

```ts
const ALLOWED_HOSTS = new Set(['app.swa.com', 'crew.swa.com']);

export function safeRedirect(url: string): string {
  try {
    const u = new URL(url, window.location.origin);
    if (u.protocol === 'javascript:' || u.protocol === 'data:') return '/';
    if (u.origin !== window.location.origin && !ALLOWED_HOSTS.has(u.host)) return '/';
    return u.pathname + u.search + u.hash;
  } catch {
    return '/';
  }
}
```

## Sensitive storage
- Auth tokens NEVER in `localStorage`. Httponly cookie OR in-memory only.
- PII NEVER in `localStorage` / `sessionStorage` unencrypted.
- For large user-state caching, use IndexedDB with explicit per-key encryption (or simply don't cache).

## CSP at server level
The Ops Suite shell should set CSP via response header (S3/CloudFront response headers policy):

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'strict-dynamic' 'nonce-<random>';
  style-src 'self' 'unsafe-inline';
  connect-src 'self' https://*.swa.com https://*.amazonaws.com;
  img-src 'self' data: https:;
  font-src 'self' data:;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
  upgrade-insecure-requests;
```

Meta-tag CSP is bypassable — flag any project relying on it.

## iframes
Any `<iframe>` requires `sandbox="..."` with the minimum capabilities. `allow-same-origin` only when the source is same-origin and trusted.

## Vulnerability sweep
`npm audit --audit-level=high` is CI-required. CRITICAL/HIGH = block (UI rule + BE checklist).
