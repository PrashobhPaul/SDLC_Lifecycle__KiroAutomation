# Frontend Code Review Checklist

Code Reviewer agent runs this. Source = full UI audit. Every finding cites file:line + tool ref + concrete fix.

## Tooling pass (T1 – T20)

```bash
mkdir -p /tmp/audit
(node_modules/.bin/tsc --noEmit 2>&1 | grep "^src/"      > /tmp/audit/t1_src.txt;
 node_modules/.bin/tsc --noEmit 2>&1 | grep "node_modules/" | head -5 > /tmp/audit/t1_node.txt) &
(npm test -- --coverage --passWithNoTests --watchAll=false   > /tmp/audit/t2.txt 2>&1) &
(npm audit --json                                            > /tmp/audit/t3.txt 2>/dev/null) &
(node_modules/.bin/eslint src --ext .ts,.tsx                  > /tmp/audit/t4.txt 2>&1) &
(node tools/find-unused-deps.js                              > /tmp/audit/t5.txt) &
(grep -r "require(" node_modules/ 2>/dev/null | grep -v "node_modules/\|@emotion" \
   | grep "^node_modules/[^/]*/index" | head -10              > /tmp/audit/t6.txt) &
(grep "transformIgnorePatterns" jest.config.js                > /tmp/audit/t7.txt) &
(grep -rn "\\|\\| true\\|exit 0" Makefile .gitlab-ci.yml 2>/dev/null > /tmp/audit/t8.txt) &
(grep -rn "console\\.\\(log\\|warn\\|error\\|info\\)" src/ --include="*.ts" --include="*.tsx" \
   | grep -v "__tests__\\|test\\|spec\\|logger\\|setupTests"   > /tmp/audit/t9.txt) &
(grep -rn "^/\\* eslint-disable \\*/" src/ --include="*.ts" --include="*.tsx" > /tmp/audit/t10.txt) &
(grep -rn "eslint-disable" src/ --include="*.ts" --include="*.tsx" \
   | grep -v "eslint-disable-next-line\\|eslint-disable-line\\|-- " | head -20 > /tmp/audit/t10b.txt) &
(grep -rn "TODO\\|FIXME\\|HACK\\|@ts-ignore" src/ --include="*.ts" --include="*.tsx" > /tmp/audit/t11.txt) &
(find src/ \\( -name "*.ts" -o -name "*.tsx" \\) | grep -v "__tests__\\|test\\|spec\\|\\.d\\.ts\\|\\.styled\\." \
   | xargs wc -l 2>/dev/null | sort -rn | head -20            > /tmp/audit/t12.txt) &
(node_modules/.bin/madge --circular src/                       > /tmp/audit/t13.txt 2>&1) &
(npx ts-prune | grep -v "test\\|spec\\|__tests__" | head -30   > /tmp/audit/t14.txt) &
(grep -rn "^const .* = new " src/ --include="*.ts" --include="*.tsx" \
   | grep -v "__tests__\\|test\\|spec\\|styled\\|\\.d\\.ts"    > /tmp/audit/t15.txt) &
(grep "services\\|!src" jest.config.js                         > /tmp/audit/t16.txt) &
(grep -n "libraryTarget\\|system\\|single-spa" webpack.config.js 2>/dev/null | head -3 > /tmp/audit/t17.txt) &
(grep -n "splitChunks\\|contenthash\\|optimization" webpack.config.js 2>/dev/null > /tmp/audit/t18.txt) &
(ls cypress/ playwright/ e2e/ 2>/dev/null || echo "NO E2E"     > /tmp/audit/t19.txt) &
(npx browserslist 2>/dev/null | head -10                       > /tmp/audit/t20.txt) &
wait
```

## Output files
- `REVIEW-metrics.md` — header, test results, coverage, tsc errors, npm audit
- `REVIEW-arch.md` — cross-file findings (ARCH-1, ARCH-2, …)
- `REVIEW-files.md` — per-file table `# | Line(s) | Severity | Category | Finding | Fix`
- `REVIEW-summary.md` — Section U (config), severity roll-up, refactor candidates, dependency risk, E2E gap

## Severity reference

| Level | Triggers |
|-------|---------|
| **CRITICAL** | Missing peer dep breaking tests/build · `dangerouslySetInnerHTML` w/o DOMPurify · secrets in `src/` · `href="javascript:"` · open-redirect · `transformIgnorePatterns` missing pkg · `npm audit` CRITICAL/HIGH · auth-protected route via UI hide only · server-side authz absent |
| **MAJOR** | tsc error · test suite failure · low/0% coverage · DRY violation · dead export · missing memo · uncovered branch · magic string · inline style · hardcoded hex/px · blanket `eslint-disable` · module-level mutable singleton · no `ErrorBoundary` · CI suppression · services excluded from coverage · `any` · prop drilling >1 · >2 `useState` · static inline style · hardcoded breakpoint · `getByTestId` · `useQuery`/`useMutation` without `loading`/`error` |

## Enforcement rules (1–28 from UI audit)

| # | Rule | Severity |
|---|------|----------|
| 1 | Every finding → exact line(s) + tool ref + concrete fix | — |
| 2 | Cross-file issues → reference every affected file | — |
| 3 | Only report what tools confirm or direct inspection proves | — |
| 4 | Write output across 4 files; write each as soon as ready | — |
| 5 | File >200 LOC / function >25 LOC → MAJOR | MAJOR |
| 6 | Pattern in >2 files → REVIEW-arch.md | — |
| 7 | Every CRITICAL → ready-to-use code fix | — |
| 8 | Coverage % from T2 only; tsc errors from T1 only; unused deps from T5 only | — |
| 9 | Single-spa MFE (`libraryTarget: "system"`) → `splitChunks` intentionally absent; do NOT flag | — |
| 10 | `\|\| true` on any CI quality gate | MAJOR |
| 11 | Blanket `/* eslint-disable */` w/o reason + ticket | MAJOR per file |
| 12 | Module-level mutable singleton | MAJOR |
| 13 | No `ErrorBoundary` in codebase | MAJOR architectural |
| 14 | `React.lazy` without both `<Suspense>` and `<ErrorBoundary>` | MAJOR per import |
| 15 | `services/**` excluded from coverage → MAJOR | MAJOR |
| 16 | Missing peer dep causing test failures | CRITICAL |
| 17 | `transformIgnorePatterns` missing a pkg that needs transform | CRITICAL |
| 18 | Enum with `eslint-disable no-unused-vars` per member → recommend `as const` | — |
| 19 | E2E suite absent | MAJOR architectural |
| 20 | `any` in production code | MAJOR per instance |
| 21 | `console.log/warn/error/info` in production code | MAJOR per file |
| 22 | Prop drilling >1 level | MAJOR |
| 23 | >2 related `useState` calls in one component | MAJOR |
| 24 | Static inline style (non-dynamic) | MAJOR per instance |
| 25 | Hardcoded hex/px/rem (not from tokens) | MAJOR per instance |
| 26 | `getByTestId` in tests instead of `getByRole`/`getByLabelText` | MAJOR per instance |
| 27 | `useQuery`/`useMutation`/`useLazyQuery` without `error` handling | MAJOR per callsite |
| 28 | `useQuery`/`useMutation`/`useLazyQuery` without `loading` handling | MAJOR per callsite |
