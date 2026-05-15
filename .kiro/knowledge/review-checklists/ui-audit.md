# UI React/TS Audit — Canonical Checklist

This is the canonical UI review checklist used by the Code Reviewer agent for
any change touching the `react-mfe` layer. Sections A–W with 28 enforcement
rules. Tools T1–T20 are run in parallel by the Code Reviewer's tooling pass.

## Severity

- **CRITICAL** — blocks G9 (pre-merge); auto-triggers G6 (reviewer-blocker-override).
- **MAJOR** — does not block by itself; rolls into per-agent AIQ scorecard.
- **MINOR** — informational; rolls into pattern-adherence axis.

## Tools (parallel)

| Tool | Purpose                                       |
|------|-----------------------------------------------|
| T1   | tsc --noEmit (strict)                         |
| T2   | eslint --max-warnings=0                       |
| T3   | vitest --coverage                             |
| T4   | playwright (smoke flows)                      |
| T5   | axe-core (a11y scan)                          |
| T6   | semgrep (security)                            |
| T7   | gitleaks (secret scan)                        |
| T8   | npm audit                                     |
| T9   | depcheck / ts-prune (dead code)               |
| T10  | madge (cycles)                                |
| T11  | webpack-bundle-analyzer (size budget)         |
| T12  | source-map-explorer (chunk sanity)            |
| T13  | lighthouse-ci (Web Vitals)                    |
| T14  | jest-axe (component-level a11y)               |
| T15  | percy (visual regression, optional)           |
| T16  | css-stats (token compliance)                  |
| T17  | graphql-codegen --check                       |
| T18  | apollo client lint (cache + fetchPolicy)      |
| T19  | knip (unused exports)                         |
| T20  | size-limit (entry-bundle gate)                |

## Rule Index (28 rules across A–W)

| #  | Section | Rule                                                           | Severity  |
|----|---------|----------------------------------------------------------------|-----------|
| 1  | A       | Feature-based folder structure respected                       | MAJOR     |
| 2  | B       | No cross-feature imports outside `/shared` or `/lib`           | MAJOR     |
| 3  | C       | TypeScript strict + noUncheckedIndexedAccess on                | CRITICAL  |
| 4  | C       | No `any` introduced                                            | CRITICAL  |
| 5  | C       | No `!` non-null assertion                                      | MAJOR     |
| 6  | D       | No hardcoded hex / px / rem; design tokens only                | MAJOR     |
| 7  | E       | No `!important` in CSS                                         | MAJOR     |
| 8  | E       | `prefers-reduced-motion` honoured                              | MAJOR     |
| 9  | F       | Lowest-scope state; `useReducer` when >2 `useState` co-evolve  | MAJOR     |
| 10 | F       | No array index as key on reorderable lists                     | MAJOR     |
| 11 | G       | Every page is `React.lazy` + has Suspense + ErrorBoundary      | CRITICAL  |
| 12 | G       | RBAC guards are server-side validated, not client-trusted      | CRITICAL  |
| 13 | H       | Bundle code-splitting respected; no inline literal explosions  | MAJOR     |
| 14 | H       | `react-window` for lists > 20 items                            | MAJOR     |
| 15 | I       | Every Apollo query: explicit `fetchPolicy` with rationale      | MAJOR     |
| 16 | J       | No empty `catch`                                               | CRITICAL  |
| 17 | J       | Network vs application errors are distinguished                | MAJOR     |
| 18 | K       | No business logic inside JSX                                   | MAJOR     |
| 19 | K       | No component with > 7 props or > 2 boolean props               | MAJOR     |
| 20 | L       | No `dangerouslySetInnerHTML` without DOMPurify                 | CRITICAL  |
| 21 | L       | No PII in `localStorage` unencrypted                           | CRITICAL  |
| 22 | M       | No `console.*` outside the logger module                       | MAJOR     |
| 23 | M       | WCAG 2.1 AA met; combobox has the 4 mandatory ARIA attrs       | CRITICAL  |
| 24 | M       | 44×44 touch targets on mobile                                  | MAJOR     |
| 25 | N       | Coverage thresholds: utils 95% / hooks 90% / components 70%    | CRITICAL  |
| 26 | N       | Query priority: getByRole > getByLabelText > getByText > id    | MAJOR     |
| 27 | O       | GraphQL: typed hooks via codegen; no string-template `gql`     | MAJOR     |
| 28 | O       | Every query handles loading + error; no N+1 in render          | MAJOR     |

(Sections P–W cover supporting concerns: peer-deps, single-spa libraryTarget,
lighthouse budgets, focus management, dark-mode, feature-detection,
transformIgnorePatterns, mock-data type match. Each rolls into per-agent AIQ.)
