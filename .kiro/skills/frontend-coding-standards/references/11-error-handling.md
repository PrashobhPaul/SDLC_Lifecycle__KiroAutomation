# Error Handling (Frontend)

## Hard rules (mapped to UI audit L)

| Rule | Severity | Source |
|------|----------|--------|
| Unhandled `async`/`await` or `.then` chain | MAJOR per instance | UI checklist L |
| Empty `catch {}` or `catch(e) {}` with no body | MAJOR | UI checklist L |
| User-facing message showing error code / stack / generic "Something went wrong" | MAJOR | UI checklist L |
| Network errors handled the same as application errors | MAJOR | UI checklist L |
| `window.alert()` / `window.confirm()` / ad-hoc inline errors | MAJOR | UI checklist L |

## Network vs application errors

```ts
// shared/lib/error-classification.ts
export type ClassifiedError =
  | { kind: 'network'; transient: boolean }
  | { kind: 'auth';    code: 'UNAUTHORIZED' | 'FORBIDDEN' }
  | { kind: 'validation'; fields: Record<string, string[]> }
  | { kind: 'application'; code: string; message: string }
  | { kind: 'unknown' };

export function classifyError(err: unknown): ClassifiedError {
  if (err instanceof TypeError && err.message.includes('fetch')) return { kind: 'network', transient: true };
  if (err instanceof ApolloError) {
    const code = err.graphQLErrors[0]?.extensions?.code;
    if (code === 'UNAUTHORIZED' || code === 'FORBIDDEN') return { kind: 'auth', code };
    if (code === 'VALIDATION') return { kind: 'validation', fields: err.graphQLErrors[0].extensions.fields ?? {} };
    if (typeof code === 'string') return { kind: 'application', code, message: err.message };
  }
  return { kind: 'unknown' };
}
```

## Toast-based feedback

```tsx
// shared/lib/toast.tsx
import { useToastStore } from './toast-store';

export const toast = {
  success: (msg: string) => useToastStore.getState().push({ kind: 'success', msg }),
  warning: (msg: string) => useToastStore.getState().push({ kind: 'warning', msg }),
  error:   (msg: string) => useToastStore.getState().push({ kind: 'error', msg }),
};
```

Every user action's failure goes through `toast.error(...)`. No `alert()` / `confirm()`.

## Actionable messages

```tsx
function presentError(err: ClassifiedError): { title: string; description: string; action?: { label: string; onClick: () => void } } {
  switch (err.kind) {
    case 'network':     return { title: 'Connection lost',     description: 'Check your network and try again.',                  action: { label: 'Retry',     onClick: refetch } };
    case 'auth':        return { title: 'Session expired',     description: 'Please sign in again to continue.',                  action: { label: 'Sign in',    onClick: signIn } };
    case 'validation':  return { title: 'Check the form',      description: 'Some fields need correction. Highlighted below.' };
    case 'application': return { title: err.code,              description: err.message };
    case 'unknown':     return { title: 'Unexpected issue',    description: 'The team has been notified.' };
  }
}
```

## Async safety

```tsx
async function onSubmit() {
  try {
    await save(form);
    toast.success('Saved');
  } catch (err: unknown) {
    logger.error('save-failed', { err });
    const present = presentError(classifyError(err));
    toast.error(present.title);
  }
}
```

Never `void save(form)` at module scope. Never bare `.then(...)` without `.catch(...)`.
