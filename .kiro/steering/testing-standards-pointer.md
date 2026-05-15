---
id: testing-standards-pointer
title: Testing Standards Pointer
version: 1
applies_to:
  - agent: test-engineer
    when: always
  - agent: developer
    when: on-keyword
    keywords: [test, testing, coverage, jest, vitest, playwright, msw]
  - agent: code-reviewer
    when: on-file-pattern
    file_globs: ["**/*.test.{ts,tsx}", "**/*.spec.{ts,tsx}", "**/__tests__/**"]
points_to:
  - kind: skill
    path: skills/frontend-coding-standards/references/15-testing.md
    label: FE testing standards (RTL, query priority, coverage targets)
  - kind: skill
    path: skills/backend-coding-standards/references/13-testing.md
    label: BE testing standards (jest.Mocked ports, aws-sdk-client-mock, coverage)
  - kind: skill
    path: skills/event-driven-aws-standards/references/04-testing-consumers.md
    label: Event-consumer testing patterns
  - kind: skill
    path: skills/vtl-resolver-standards/references/03-testing-vtl.md
    label: VTL simulator testing
token_budget:
  load_on_activation_tokens: 1500
  cumulative_session_tokens: 6000
governance:
  pii_class: none
  data_scope: [local]
  hitl_required: false
checksum: pending-linter-recompute
---
# Testing Standards

One pointer for the full test pyramid across CALM layers. Coverage targets per layer:
domain ≥95%, application ≥90%, adapters ≥80%, components 70%, utils 95%, hooks 90%.
Query priority `getByRole > getByLabelText > getByText > getByTestId` (last resort).
Test data factories pure and deterministic. AppSync simulator used for VTL goldens.
