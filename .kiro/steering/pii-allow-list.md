---
id: pii-allow-list
title: PII Allow-List Pointer
version: 1
applies_to:
  - agent: feature-builder
    when: always
  - agent: developer
    when: always
  - agent: test-engineer
    when: always
  - agent: code-reviewer
    when: always
  - agent: documentation
    when: always
points_to:
  - kind: knowledge
    path: knowledge/security/pii-allow-list.yml
    label: Crew domain PII fields permitted with redaction class
  - kind: skill
    path: skills/guardrails-io-sanity/references/03-pii-redaction.md
    label: Redaction rules and regex patterns
token_budget:
  load_on_activation_tokens: 600
  cumulative_session_tokens: 3000
governance:
  pii_class: guarded
  data_scope: [local]
  hitl_required: false
  on_unredacted_match: refuse
checksum: pending-linter-recompute
---
# PII Allow-List Pointer

The crew domain handles PII (employee IDs, FMLA records, schedules). This pointer
tells every agent where the allow-list lives and what redactor to invoke on input
and output. The redactor runs as part of the I/O sanity guardrail; agents never
emit unredacted PII into AIQ records, logs, or handoff envelopes.
