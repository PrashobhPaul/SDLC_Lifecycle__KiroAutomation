# Component Library & Styling

## Hard rules (mapped to UI audit E)

| Rule | Severity | Source |
|------|----------|--------|
| Native `<button>` / `<input>` / `<select>` built alongside design system | MAJOR | UI checklist E |
| `!important` anywhere | MAJOR per instance | UI checklist E [T4] |
| Deep descendant selector targeting library internals (`& > div > span`) | MAJOR | UI checklist E |
| Hardcoded hex / px / rem (not from tokens) | MAJOR per instance | UI rule 25 |
| Light-only color values (no theme-provider/CSS-var path) | MAJOR | UI checklist E |
| Runtime config baked into bundle (not from `window.__APP_CONFIG__`) | MAJOR | UI checklist E |
| Library ARIA props not forwarded to underlying DOM | MAJOR | UI checklist E |
| Static inline style on a non-dynamic value | MAJOR per instance | UI rule 24 |
| Unused styled components | MAJOR | UI checklist E [T14] |
| Animation of `width` / `height` / `top` / `left` / `margin` / `padding` | MAJOR | UI checklist E |
| `prefers-reduced-motion` not respected | MAJOR | UI checklist E |

## Design tokens

Single source of truth at `src/theme/tokens.ts`. Tokens flow into the theme provider; consumers reach
them via `useTheme()` or styled components.

```ts
// src/theme/tokens.ts
export const tokens = {
  color: {
    bg:          { default: 'var(--color-bg-default)',    subtle: 'var(--color-bg-subtle)' },
    text:        { default: 'var(--color-text-default)',  muted:  'var(--color-text-muted)' },
    accent:      { default: 'var(--color-accent-default)', strong: 'var(--color-accent-strong)' },
    semantic:    { error: 'var(--color-semantic-error)',  success: 'var(--color-semantic-success)' },
  },
  space:    { 0: '0', 1: '4px', 2: '8px', 3: '12px', 4: '16px', 6: '24px', 8: '32px' },
  radius:   { sm: '4px', md: '8px', lg: '12px', round: '9999px' },
  font: {
    family: { sans: 'var(--font-family-sans)', mono: 'var(--font-family-mono)' },
    size:   { xs: '12px', sm: '14px', md: '16px', lg: '18px', xl: '24px' },
    weight: { regular: 400, medium: 500, bold: 700 },
  },
  shadow:   { sm: '0 1px 2px rgba(0,0,0,0.04)', md: '0 4px 8px rgba(0,0,0,0.08)' },
  motion:   { fast: '120ms', base: '180ms', slow: '260ms', easing: 'cubic-bezier(0.2, 0, 0, 1)' },
} as const;
```

## Theme provider

```tsx
// src/theme/ThemeProvider.tsx
import { createContext, useContext, type PropsWithChildren } from 'react';
import { tokens } from './tokens';

const ThemeCtx = createContext(tokens);

export function ThemeProvider({ children }: PropsWithChildren): JSX.Element {
  return <ThemeCtx.Provider value={tokens}>{children}</ThemeCtx.Provider>;
}

export const useTheme = (): typeof tokens => useContext(ThemeCtx);
```

## Dark mode

CSS variables define both light and dark palettes; the theme provider toggles `data-theme` on the root.

```css
:root[data-theme="light"] { --color-bg-default: #fff; --color-text-default: #0a0a0a; }
:root[data-theme="dark"]  { --color-bg-default: #0a0a0a; --color-text-default: #fafafa; }
```

## Animations

```ts
// Animate transform / opacity only
const fadeUp = keyframes`
  from { transform: translateY(8px); opacity: 0; }
  to   { transform: translateY(0);   opacity: 1; }
`;

// Respect prefers-reduced-motion
const wrapper = css`
  animation: ${fadeUp} ${tokens.motion.base} ${tokens.motion.easing} both;
  @media (prefers-reduced-motion: reduce) {
    animation: none;
    opacity: 1;
  }
`;
```

## Runtime config

```ts
// src/lib/config.ts
declare global {
  interface Window {
    __APP_CONFIG__?: { graphqlUrl: string; environment: 'dev1' | 'golden-dev' | 'qa' | 'stage' | 'prod' };
  }
}

export function getRuntimeConfig() {
  const cfg = window.__APP_CONFIG__;
  if (!cfg) throw new Error('Runtime config missing — server must inject window.__APP_CONFIG__');
  return cfg;
}
```

`.env.example` lists every key referenced in `src/`.
