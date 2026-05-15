# RACI — AI vs Human

Direct mapping to the roadshow slide ("Roles are not eliminated — they become more focused").

| Concern | AI Handles | Human Owns |
|--------|------------|------------|
| Feature structuring from raw inputs | Feature Builder drafts Epics, Stories, AC | PO validates intent and priority before G1 |
| Story decomposition into tasks per layer | Feature Builder | PO/Tech-Lead review before G1 |
| Code generation across layers | Developer | Tech-Lead approves plan at G2; reviews diffs |
| Pattern enforcement / coding standards | Developer + Code Reviewer | Tech-Lead overrides via G6 with rationale |
| Test pyramid generation | Test Engineer | QA-Lead approves strategy at G5 |
| Static review (TS, ESLint, GraphQL diff) | Code Reviewer | Tech-Lead approves blocker overrides at G6, merges at G9 |
| IaC review (tfsec, tflint, IAM) | Code Reviewer | SRE/Cyber approves any IAM expansion |
| Doc generation + stale-doc detection | Documentation | Tech-writer approves stale-doc decisions at G8 |
| Tool calls outside SWA scope | Agent **refuses** | Cyber approves new scope before agent can use it |
| Final merge | None — never auto-merge | Tech-Lead pulls the trigger after G9 |

## Accountability principle
"AI recommends; humans approve. Accountability stays with people."
- AI proposes a diff, a finding, a decomposition.
- A human approves at the relevant gate.
- The decision record references both: agent run id + human approver.
