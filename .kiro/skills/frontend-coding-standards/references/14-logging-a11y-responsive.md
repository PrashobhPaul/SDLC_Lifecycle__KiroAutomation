# Logging, Accessibility, Responsiveness

## Logging — UI audit O

| Rule | Severity | Source |
|------|----------|--------|
| `console.log/warn/error/info` outside the logger module | MAJOR per file | UI rule 21 [T9] |
| Log payload containing PII (names, IDs, DOB, emails) | MAJOR per field | UI checklist O |
| Log payload containing secret / auth token | CRITICAL | UI checklist O |
| Free-form string log message instead of structured key-value object | MAJOR | UI checklist O |

```ts
// src/lib/logger/index.ts
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

function emit(level: LogLevel, message: string, context: Record<string, unknown> = {}): void {
  const sanitized = redactPii(context);
  const entry = { ts: new Date().toISOString(), level, message, ...sanitized };
  if (typeof window !== 'undefined' && window.__APP_CONFIG__?.environment !== 'prod') {
    // Visible in dev console only
    // eslint-disable-next-line no-console -- only logger module is allowed
    console[level === 'debug' ? 'log' : level](entry);
  }
  void shipToBackend(entry);
}

export const logger = {
  debug: (m: string, c?: Record<string, unknown>) => emit('debug', m, c),
  info:  (m: string, c?: Record<string, unknown>) => emit('info',  m, c),
  warn:  (m: string, c?: Record<string, unknown>) => emit('warn',  m, c),
  error: (m: string, c?: Record<string, unknown>) => emit('error', m, c),
};
```

## Accessibility — UI audit P (WCAG 2.1 AA)

| Rule | Severity | Source |
|------|----------|--------|
| Interactive element without `aria-label` or visible label | MAJOR | UI checklist P |
| Combobox missing 4 attrs (`aria-haspopup="listbox"`, `aria-expanded`, `aria-controls`, `aria-activedescendant`) | MAJOR | UI checklist P |
| `<div>`/`<span>` where `<button>` / `<nav>` / `<main>` / `<section>` / `<article>` applies | MAJOR | UI checklist P |
| Element not keyboard-operable (Tab, Enter, Space, Escape, arrows) | MAJOR | UI checklist P |
| Focus not trapped inside open modal / not restored to trigger on close | MAJOR | UI checklist P |
| State conveyed by color alone | MAJOR | UI checklist P |
| `<img>` without `alt` | MAJOR | UI checklist P |
| `<input>` without associated `<label>` (placeholder is not a label) | MAJOR | UI checklist P |
| Error not linked via `aria-describedby` | MAJOR | UI checklist P |
| Dynamic content update without `aria-live="polite"` | MAJOR | UI checklist P |
| Async button without `aria-busy={isLoading}` | MAJOR | UI checklist P |
| Paginated table row without `aria-rowindex` | MAJOR | UI checklist P |
| Page missing skip-navigation link | MAJOR | UI checklist P |
| Touch target smaller than 44 × 44 px | MAJOR | UI checklist P |

## Combobox required attrs

```tsx
<div role="combobox"
     aria-haspopup="listbox"
     aria-expanded={open}
     aria-controls={listboxId}
     aria-activedescendant={activeId}>
  <input aria-autocomplete="list" />
</div>
<ul role="listbox" id={listboxId}>...</ul>
```

## Focus management

```tsx
function Modal({ open, onClose, children }: ModalProps) {
  const triggerRef = useRef<HTMLElement | null>(document.activeElement as HTMLElement);
  const dialogRef  = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!open) return;
    const first = dialogRef.current?.querySelector<HTMLElement>('[autofocus], button, [tabindex]');
    first?.focus();
    return () => triggerRef.current?.focus();
  }, [open]);

  // ...
}
```

## Responsiveness — UI audit Q

| Rule | Severity | Source |
|------|----------|--------|
| `npx browserslist` mismatch with stated browser support | MAJOR | UI checklist Q [T20] |
| Hardcoded breakpoint (`@media (max-width: 768px)`) not from tokens | MAJOR | UI checklist Q |
| Browser API used without feature detection | MAJOR | UI checklist Q |

```ts
// src/theme/breakpoints.ts
export const breakpoints = {
  sm: '480px',
  md: '768px',
  lg: '1024px',
  xl: '1440px',
} as const;

export const media = {
  sm: `@media (min-width: ${breakpoints.sm})`,
  md: `@media (min-width: ${breakpoints.md})`,
  lg: `@media (min-width: ${breakpoints.lg})`,
  xl: `@media (min-width: ${breakpoints.xl})`,
} as const;
```

```ts
// Feature detection
if ('clipboard' in navigator && 'writeText' in navigator.clipboard) {
  await navigator.clipboard.writeText(text);
} else {
  // Fallback path
}
```
