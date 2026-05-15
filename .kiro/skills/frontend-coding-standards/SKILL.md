---
name: frontend-coding-standards
description: >
  Frontend (React TypeScript) coding standards skill for the SWA CALM platform. Activate on any task
  that creates, modifies, or reviews .ts / .tsx files under src/. Encodes feature-based architecture,
  peer-deps + transformIgnorePatterns gotchas, strict TypeScript, design tokens, state management,
  routing, performance, data fetching, suspense + ErrorBoundary, error handling, security, logging,
  a11y, responsive design, testing, GraphQL with codegen, and the full UI audit checklist (sections
  A–W, 28 enforcement rules) used by the Code Reviewer agent.
---

# Frontend Coding Standards — Skill

## How to use this skill

| Concern | Reference |
|---------|-----------|
| Architecture (feature-based folders, /shared, /lib boundaries) | `references/01-architecture.md` |
| Peer-deps, transformIgnorePatterns, jest config gotchas | `references/02-peer-deps.md` |
| TypeScript flags (strict, noUncheckedIndexedAccess, exactOptionalPropertyTypes) | `references/03-typescript.md` |
| Component library + design tokens + styling | `references/04-component-library-styling.md` |
| State management (lowest scope, prop-drilling, useReducer threshold) | `references/05-state-management.md` |
| Routing & navigation (React.lazy, RBAC server-side, 404) | `references/06-routing-navigation.md` |
| Performance (single-spa libraryTarget, splitChunks, contenthash, react-window) | `references/07-performance.md` |
| Data fetching & caching (fetchPolicy, optimistic, AbortController) | `references/08-data-fetching-caching.md` |
| Data processing (parseLocalDate, mappers pure) | `references/09-data-processing.md` |
| Suspense & ErrorBoundary (every React.lazy needs both) | `references/10-suspense-error-boundaries.md` |
| Error handling (no empty catch, network vs app, toast system) | `references/11-error-handling.md` |
| Logic & SOLID (SRP, max 7 props, no >2 booleans, conditional nesting) | `references/12-logic-solid.md` |
| Security (DOMPurify, no secrets, server-side authz, CSV injection, CSP) | `references/13-security.md` |
| Logging, a11y, responsive (combobox 4 attrs, 44x44, focus mgmt) | `references/14-logging-a11y-responsive.md` |
| Testing (RTL, query priority, coverage thresholds, no snapshot for logic) | `references/15-testing.md` |
| GraphQL (codegen typed hooks, no string templates, every callsite handles loading+error) | `references/16-graphql.md` |
| Code review checklist (UI audit A–W, 28 rules, T1–T20) | `references/17-code-review-checklist.md` |

## Always-on principles

- Single-spa MFE: `libraryTarget: "system"` and `splitChunks` intentionally absent — do not flag.
- Standard SPA: `splitChunks: { chunks: 'all' }` and `[contenthash:8]` required (UI rules in H).
- Every interactive element uses the design system component, not a native element built alongside.
- Every `React.lazy` has BOTH `<Suspense>` and `<ErrorBoundary>` ancestors (UI rule 14).
- Every `useQuery` / `useMutation` / `useLazyQuery` handles BOTH `loading` AND `error` (UI rules 27/28).
- No `dangerouslySetInnerHTML` without DOMPurify (UI rule N — CRITICAL).
- No `any` in production (UI rule 20 — MAJOR per instance).
- Prop drilling >1 level → MAJOR (UI rule 22).
- >2 related `useState` calls → `useReducer` (UI rule 23).
- `getByTestId` only as last resort (UI rule 26).
