---
id: documentation-standards-pointer
title: Documentation Standards Pointer
version: 1
applies_to:
  - agent: documentation
    when: always
  - agent: developer
    when: on-keyword
    keywords: [tsdoc, jsdoc, readme, changelog, api-docs]
  - agent: code-reviewer
    when: on-file-pattern
    file_globs: ["**/*.md", "docs/**", "**/README.md", "CHANGELOG.md"]
points_to:
  - kind: skill
    path: skills/graphql-appsync-standards/references/04-codegen-and-client.md
    label: GraphQL API doc generation
  - kind: skill
    path: skills/terraform-aws-standards/references/01-module-structure.md
    label: terraform-docs README format
  - kind: skill
    path: skills/vtl-resolver-standards/references/04-review-checklist.md
    label: VTL operation→data-source mapping doc
token_budget:
  load_on_activation_tokens: 600
  cumulative_session_tokens: 3000
governance:
  pii_class: low
  data_scope: [local]
  hitl_required: false
checksum: pending-linter-recompute
---
# Documentation Standards

TSDoc on every public export. Component prop tables + a11y notes auto-generated. GraphQL
API docs from schema introspection (regen on schema save via hook). VTL resolver docs map
operation → data source. terraform-docs generates module READMEs. Changelog follows
Conventional Commits. Diagrams emit BOTH Mermaid (display) and drawio (editable).
