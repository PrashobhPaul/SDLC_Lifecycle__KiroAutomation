# Steering Pointer Schema v1

Locked schema for `.kiro/steering/*.md` files. Roadmap commitment: **Sprint 1 — KB pointer pattern v1**.

Steering files MUST be pointer-only. Bulk content lives in `.kiro/knowledge/` or `.kiro/skills/<skill>/references/`. Steering pointers tell agents *where to look* and *when*; they never carry the content itself.

---

## File anatomy

```yaml
---
id: <kebab-case-id>                       # required, unique across .kiro/steering
title: <human-readable>                   # required
version: 1                                # required, integer
applies_to:                               # required, list, controls activation
  - agent: <feature-builder|developer|test-engineer|code-reviewer|documentation|*>
    when: <always|on-keyword|on-file-pattern|on-tool-call|on-handoff-receive>
    keywords: [<words>]                   # required when when=on-keyword
    file_globs: [<globs>]                 # required when when=on-file-pattern
points_to:                                # required, list of resolvable refs
  - kind: knowledge|skill|schema|hook|template
    path: <repo-relative-path>
    label: <short label shown to operator>
token_budget:                             # required, enforced by validaite/budget hook
  load_on_activation_tokens: <int>        # cap for what activation pulls into context
  cumulative_session_tokens: <int>        # cap across the agent's session
governance:                               # required
  pii_class: none|low|medium|high
  data_scope: [confluence|jira|figma|gitlab|local|none]
  hitl_required: true|false
checksum: <sha256-of-pointed-targets>     # required, recomputed by linter
---
# <Title>

<one-paragraph human-readable rationale — why this pointer exists, when it activates,
what the agent must NOT do without it. No bulk content. Max ~80 lines.>
```

---

## Validation rules (enforced by `pointer-schema-linter` hook)

1. Frontmatter is valid YAML; every required key is present.
2. Body is ≤ 80 lines, contains no fenced code blocks of language `<reference|content>`.
3. Every `points_to[].path` resolves on disk.
4. `checksum` matches sha256 over concatenation of all `points_to[]` target file bodies.
5. `applies_to[].agent` is one of the five canonical agent names or `*`.
6. `token_budget.load_on_activation_tokens` ≤ `token_budget.cumulative_session_tokens`.
7. No `applies_to[].when=always` unless `governance.hitl_required=false` AND
   `token_budget.load_on_activation_tokens` ≤ 1500. (Always-loaded files must be small.)
8. If `governance.pii_class` is medium or high, `governance.hitl_required` MUST be true.

The linter exits non-zero on any violation. Pre-merge hook blocks the MR.

---

## Why the schema exists (operator note)

- Token economy: agents shouldn't re-read the entire steering set on every turn.
- Auditability: every activation is traceable to a pointer with a checksum.
- Decoupling: skills, knowledge, schemas evolve independently of the steering surface.
- Governance: PII and data-scope live with the pointer, not with the bulk file.

---

## Versioning

Breaking changes to this schema bump the major. Pointers below current schema version
are accepted with a warning until the next minor; after that the linter rejects them.

Current: **v1** — locked 2026-04-26.
