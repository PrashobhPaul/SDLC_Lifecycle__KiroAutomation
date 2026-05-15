# Existing Project Checklist

Apply BEFORE any new code is written. The Developer agent runs through this on Phase 1 (Codebase Analysis).
The goal: match working patterns, do not impose new-project standards on a sound codebase.

## Detection
- [ ] `package.json` present → identify `dependencies` and `devDependencies`
- [ ] `tsconfig.json` present → capture `strict`, `target`, `moduleResolution`, `noUncheckedIndexedAccess`
- [ ] Folder layout examined: `src/domain/`, `src/application/`, `src/adapters/{inbound,outbound}/`, `src/ports/` (or alternative structures)
- [ ] AWS SDK in use: v2 vs v3 (BE rule 14 says v3, but match the project if v2 is pervasive — flag for migration)
- [ ] Logger in use: `@aws-lambda-powertools/logger`, `pino`, `winston`, or hand-rolled
- [ ] Test framework: `jest` vs `vitest` vs `node:test`
- [ ] Build: `esbuild`, `tsup`, `webpack`, raw `tsc`

## Match-then-improve
1. If the project already uses Powertools, continue with Powertools — do not introduce a parallel logger.
2. If the project already uses Vitest, generate tests in Vitest — do not switch to Jest.
3. If the project uses a hand-rolled DI container, use it — do not introduce InversifyJS.
4. If the project uses Zod for validation, use Zod — do not introduce Joi.
5. If the project uses a different folder split (e.g., `src/api/`, `src/services/`, `src/db/`), MATCH it for the new files and propose migration as a separate ticket.

## When to deviate
- A pattern violates a Phase-1 BE-audit CRITICAL rule (e.g., `@aws-sdk` import in domain layer): propose a fix via Code Reviewer, not via silent re-architecture.
- A pattern is technically working but is being deprecated (e.g., AWS SDK v2): match for the immediate change, propose a migration.

## Anti-patterns
- "Big-bang" rewrite to enforce standards — never propose this as part of a feature.
- Adding two libraries that overlap (e.g., Pino + Powertools logger).
- Folder rename mid-feature.

## Output
The Developer agent records the detected stack in the plan (G2 evidence). The plan explicitly states which patterns are being matched and which deviations are deliberate, with a one-line rationale per deviation.
