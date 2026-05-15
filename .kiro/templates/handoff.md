# Handoff — Human-Readable Mirror

This template is the human-readable mirror of the JSON envelope at
`schemas/handoff-envelope.schema.json`. The agent emits both: the JSON for the
machine, this Markdown for review.

## Handoff <handoff_id>

| Field           | Value                                                |
|-----------------|------------------------------------------------------|
| Feature         | <feature-id> — <feature-title>                       |
| Emitted by      | <agent>                                              |
| Emitted at      | <ISO-8601 UTC>                                       |
| Trace ID        | <trace_id>                                           |
| Schema version  | <envelope_version>                                   |

### Scope

**In scope:**
- <bullet>

**Out of scope:**
- <bullet>

**Layers touched:** react-mfe / graphql-schema / node-resolver / vtl-resolver /
terraform-iac (delete those that don't apply)

### Context

- Epics: <links>
- Stories: <links>
- Acceptance criteria: <link to AC document>
- Open questions: <count> (see `question-back` record if > 0)

### Artefacts

| Class      | Path                                              |
|------------|---------------------------------------------------|
| Specs      | <path>                                            |
| Code       | <path>                                            |
| Tests      | <path>                                            |
| Infra      | <path>                                            |
| Docs       | <path>                                            |
| Diagrams   | <mermaid-path> + <drawio-path> (BOTH required)    |

### HITL gate state

| Gate | Status     | Decision recorded by | Timestamp |
|------|------------|----------------------|-----------|
| G1   | approve / waived / pending  | …            | …         |

### Guardrail verdict

- Input sanity: pass / refuse-and-halt
- Output sanity: pass / refuse-to-emit
- PII redaction: pass / fail
- Secret scan: pass / fail
- Pointer schema: pass / fail

### Async metadata

- Queue: <q.handoff>
- Retries used: <0–3>
- Fallback path taken: <none / write-local / etc>
