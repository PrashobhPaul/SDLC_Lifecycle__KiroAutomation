# CALM Architecture — Construction Agent Loop Model

CALM is the five-agent construction loop that takes a feature from PO refinement
through to production-ready code, tests, infrastructure, review, and documentation.

## Five Layers (output classes)

The system produces artefacts across five layers. Every agent must declare which
layers its output touches, so that the right standards are activated.

1. **react-mfe** — single-spa MFE built and deployed to S3 in the Ops Suite AWS
   account. Authoritative skill: `frontend-coding-standards`.
2. **graphql-schema** — AppSync GraphQL schema definitions. Authoritative skill:
   `graphql-appsync-standards`.
3. **node-resolver** — Lambda resolvers (TypeScript on `nodejs22.x`, arm64).
   Authoritative skill: `backend-coding-standards`.
4. **vtl-resolver** — VTL mapping templates for direct AppSync→DynamoDB resolvers
   and pipeline functions. Authoritative skill: `vtl-resolver-standards`.
5. **terraform-iac** — All AWS infrastructure. Authoritative skill:
   `terraform-aws-standards`.

## Five Agents

| Order | Agent             | Primary Output Layers                              |
|-------|-------------------|----------------------------------------------------|
| 1     | Feature Builder   | epics, stories, acceptance criteria (Gherkin)      |
| 2     | Developer         | react-mfe, graphql-schema, node-resolver, vtl, IaC |
| 3     | Test Engineer     | unit/integration/VTL goldens/E2E (per layer)       |
| 4     | Code Reviewer     | review-summary, review-files, review-arch, metrics |
| 5     | Documentation     | TSDoc, README, ADRs, diagrams (Mermaid + drawio)   |

Agents can be invoked in order **or independently**. A user starting at Developer
or Test Engineer with only a Jira ticket as input must succeed.

## Handoff Contract

Each agent emits a **handoff envelope** to its successor (schema:
`handoff-envelope.schema.json`). Envelopes carry context, scope, artefact pointers,
HITL gate states, guardrail verdict, and observability IDs. Downstream agents
refuse to start on an invalid envelope.

## Question-Back Model

If an agent cannot proceed deterministically, it emits a Question-Back record
(schema: `question-back.schema.json`). The orchestrator surfaces the questions to
the user; only after the user answers (with marker `"answered"`) does the agent
resume. Question-Backs are tracked per-feature to prevent silent re-asking.

## HITL Gates (9)

| ID  | Gate                                | Owning agent       | Hard rule          |
|-----|-------------------------------------|--------------------|--------------------|
| G1  | scope-approval                      | feature-builder    | literal `approve`  |
| G2  | plan-approval                       | developer          | literal `approve`  |
| G3  | schema-change                       | developer          | breaking change    |
| G4  | iac-change                          | developer          | destructive plan   |
| G5  | test-strategy                       | test-engineer      | literal `approve`  |
| G6  | reviewer-blocker-override           | code-reviewer      | CRITICAL finding   |
| G7  | self-correction-cap                 | developer          | cap = 3 attempts   |
| G8  | doc-stale (non-blocking)            | documentation      | drift detected     |
| G9  | pre-merge                           | code-reviewer      | gates G1–G8 status |

---

## 2026-05-14 Architecture Walkthrough — additions

The 2026-05-14 CALM walkthrough confirmed the following constraints. Full meeting
record: `knowledge/architecture/architecture-walkthrough-2026-05-14.md`.

### Account split

CALM is backed by **two AWS accounts**, not one:

- **Crew Core** — DynamoDB-backed Crew Member profile and sensitive PII.
- **Calm / DHP** — Aurora PostgreSQL for Calm-domain data (FMLA, leave events).

A separate **Ops Suite** account hosts the single-spa MFE bundles in S3.

### AppSync as sole gateway

Every UI → backend call goes through AppSync. There is **no path from the MFE to
the DynamoDB or Aurora stores that does not transit a GraphQL query/mutation**.
Each query/mutation resolves to either a VTL template or a Lambda resolver.

### Sensitive PII rule (HARD)

Sensitive PII **never transits Kafka**. Topics carry identifiers; consumers that
need PII fetch it from the dedicated Crew-Member GraphQL API.

Full rules: `knowledge/swa-stack/data-classification.md`.

### Promotion path

`DEV-1 → Golden DEV → QA → Prod`. **AI agents may target DEV-1 only.** All
promotions past DEV-1 are human actions through the SWA GitLab CI pipeline.

Full path: `knowledge/swa-stack/deployment-flow.md`.
