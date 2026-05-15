# Steering Bulk — Reading Order

Steering pointers under `.kiro/steering/` are intentionally thin (frontmatter +
one paragraph). The bulk content they point to lives here in `knowledge/` and
under `skills/<skill-name>/references/`.

Recommended reading order for a fresh reviewer / new joiner:

1. `architecture/calm-architecture.md` — what CALM is.
2. `swa-stack/aws-stack.md` — what we are building on.
3. `swa-stack/bounded-contexts.md` — domain ownership.
4. `swa-stack/multi-region.md` — DR posture.
5. `calm/scope-boundaries.md` — what is in / out of scope for Phase-1.
6. `security/red-lines.md` — what every agent must refuse.
7. `review-checklists/ui-audit.md` — UI rules.
8. `review-checklists/be-lambda-audit.md` — BE rules.
9. `product/crew-rest-violation-alert.md` — the pilot feature.

Token-budget guidance: pointers carry the budget. A pointer asks the orchestrator
to load only the section that matches the current task — never the whole file.
