# Schema Review & Breaking-Change Detection

## Tooling
```bash
npx graphql-inspector diff schema/<prev>.graphql schema/<head>.graphql --rule suppressRemovalOfDeprecatedField
```

## Breaking changes (BLOCK at G3)
- Type or field removed
- Required argument added
- Field type narrowed (`String!` → `Int!`)
- Optional → required transition
- Nullable → non-null on a return type
- Enum value removed
- Union/Interface member removed

## Non-breaking (allowed without G3)
- New optional field
- New type
- New optional argument
- Field marked `@deprecated`
- Description added/updated

## Override path
If a breaking change is intentional and consumers are coordinated:
1. File a G6 review-blocker-override referencing the consumer audit.
2. Set an explicit expiry (default 7 days; revisit at next sprint).
3. Document in `CHANGELOG.md` with a `MIGRATION:` prefix.

## CI integration
Pre-merge hook (`.kiro/hooks/pre-merge/graphql-breaking-change.json`) runs `graphql-inspector` against
the merge target. Any breaking change without a documented override fails the gate.

## Reviewer report fields
- Diff summary
- Breaking changes table (rule, location, severity)
- Affected operations (queries/mutations/subscriptions that depend on removed/changed fields)
- Affected consumers (apps, dashboards, MFEs known to query the field)
- Suggested deprecation path
