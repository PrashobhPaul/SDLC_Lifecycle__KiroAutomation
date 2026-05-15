# Acceptance Criteria — Gherkin Template

Use this template inside the Feature Builder's output for every user story.

## Story <id>: <title>

**Actors:** <user role>
**Preconditions:** <state assumed before scenario>

### Scenario: <happy path label>

```gherkin
GIVEN <initial state>
  AND <additional context>
WHEN <action by actor>
THEN <observable outcome>
  AND <secondary outcome or side effect>
```

### Scenario: <edge case label>

```gherkin
GIVEN <initial state>
WHEN <action under edge condition>
THEN <expected behaviour>
  AND <error/audit trail expectation>
```

## Rules
- One Gherkin block per scenario; do not concatenate scenarios.
- Mention the layer(s) impacted at the top of the story (react-mfe / graphql /
  node-resolver / vtl / terraform).
- Mention the HITL gates that will fire (typically G1 always; G3/G4 conditional).
