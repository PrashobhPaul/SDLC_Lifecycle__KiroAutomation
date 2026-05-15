# Peer Deps, transformIgnorePatterns, jest config gotchas

## Section A of UI audit — runs first, blocks everything else

| Rule | Severity | Source |
|------|----------|--------|
| Package imported in `src/` not in `package.json` | CRITICAL | UI rule 16 [T5] |
| Peer dep of UI lib not installed (Cannot find module at runtime) | CRITICAL | UI rule 16 [T6] |
| `@emotion/react` + `@emotion/styled` missing when CSS-in-JS used | CRITICAL | UI rule 16 |
| `jest.transformIgnorePatterns` missing a package the UI lib imports internally | CRITICAL | UI rule 17 [T7] |
| `skipLibCheck: true` masking real version incompatibilities | MAJOR | UI rule (A) [T1] |

## Why these are CRITICAL
- A missing peer dep does not always fail at install-time; it fails at runtime/test-time with `Cannot find module '<peer>'`.
- A missing transformIgnorePatterns entry causes Jest to try to parse ESM-shipping packages with the CJS transformer; it fails with a syntax error that looks like a code bug.
- Both blow up the test results in T2 and make all downstream findings unreliable.

## Detection
- T5: `cat package.json | grep deps | xargs grep src/` — every dep imported at least once.
- T6: `grep "require(" node_modules/<ui-lib>/index.cjs` — peer deps the lib expects at runtime.
- T7: `grep "transformIgnorePatterns" jest.config.js`.

## Fix patterns

```jsonc
// jest.config.js
{
  "transformIgnorePatterns": [
    "node_modules/(?!(<ui-lib>|<another-esm-only-pkg>|@swa/.*)/)"
  ]
}
```

```jsonc
// package.json
{
  "peerDependencies": {
    "@emotion/react": "^11.11.0",
    "@emotion/styled": "^11.11.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  }
}
```

## skipLibCheck gotcha
- `skipLibCheck: true` is fine, BUT the audit must also run `tsc --noEmit` and capture the node_modules errors so you know what is being masked.
- Code Reviewer reports the masked errors separately from src errors so they don't drown out the real findings.
