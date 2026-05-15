# CALM Phase-1 Scope Boundaries

## In scope (Phase-1)

- Five-agent CALM construction loop (Feature Builder, Developer, Test Engineer,
  Code Reviewer, Documentation).
- Pilot feature: **Crew Rest Violation Alert (FAA Part 117)**.
- Tooling: Confluence, Jira, Figma — read-only by default; Jira write-back permitted
  only after G1 approval. GitLab read-only; no automated push to protected branches.
- Live SWA stack reach: AppSync schema simulation, Lambda resolver scaffolding,
  Terraform planning (no apply), VTL goldens.
- ValidAIte harness: AIQ scorecards, golden-run set, governance metrics.
- Four pillars: Governance & Ethics, Observability, Guardrails & I/O Sanity,
  Async Orchestration with retries and fallback.

## Explicitly out of scope (Phase-1)

- Production Kafka writes by any agent.
- `terraform apply` against any environment by any agent.
- Direct production AWS console actions.
- Account creation / IAM principal creation by any agent.
- Cross-context DB access not mediated by the owning GraphQL API.
- Automated merge to `main` without G9 approval.
- Replacing human SMEs in calibration scoring.