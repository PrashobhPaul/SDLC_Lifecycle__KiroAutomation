# Dependency & Config Hygiene

## Hard rules (mapped to BE audit)

| Rule | Severity | Source |
|------|----------|--------|
| Unused deps in `package.json` | MAJOR per dep | BE rule (J) |
| `devDependencies` vs `dependencies` mis-categorised | MAJOR | BE checklist J |
| `package-lock.json` / `yarn.lock` not committed | MAJOR | BE checklist J |
| ESLint missing `@typescript-eslint`, `no-explicit-any`, `no-unused-vars`, `complexity` | MAJOR | BE checklist J |
| ESLint `complexity` rule > 15 | MAJOR | BE rule 41 |
| Committed `.env` file | CRITICAL | BE checklist J |
| `.env.example` missing | MAJOR | BE checklist J |
| `node_modules`, `dist`, `.terraform`, `*.backup`, `debug.json` not in `.gitignore` | MAJOR per file | BE rule 45 |
| Pre-commit hooks absent (lint, typecheck, secret scan) | MAJOR | BE checklist J |
| Bundle > 50 MB zipped | MAJOR | BE rule 64 |
| Renovate / Dependabot not configured | MAJOR | BE rule 67 |
| Semantic release not configured | MAJOR | BE checklist J |
| `sonar.coverage.exclusions` excludes `domain/` or `application/` | MAJOR | BE checklist J |
| CDN tarball URL in `package.json` | MAJOR | BE checklist J |
| GPL/AGPL license in commercial product | MAJOR | BE checklist J |

## Required ESLint config

```jsonc
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint", "import", "unicorn"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking"
  ],
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/no-non-null-assertion": "error",
    "@typescript-eslint/no-floating-promises": "error",
    "@typescript-eslint/await-thenable": "error",
    "complexity": ["error", 15],
    "max-lines-per-function": ["error", 25],
    "max-lines": ["error", 200],
    "no-console": "error",
    "no-shadow": "off",
    "@typescript-eslint/no-shadow": "error",
    "import/no-cycle": "error"
  }
}
```

## Pre-commit hooks (Husky + lint-staged)

```json
{
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"],
    "*.tf":  ["terraform fmt"],
    "*":     ["gitleaks protect --staged --redact"]
  }
}
```

## Bundling
- `esbuild` with tree-shaking and minification.
- `external` for `@aws-sdk/*` (provided by Lambda runtime since Node 20).
- Output bundle size reported in CI; fail at >50 MB zipped.
