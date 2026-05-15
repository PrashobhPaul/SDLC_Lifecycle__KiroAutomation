# Data-Scope Allow-Listing

## Purpose
Every Confluence / Jira / Figma / GitLab read is bounded by an explicit SWA-Cyber-approved scope.
Out-of-scope reads are refused at the agent boundary, not at the tool boundary.

## Allow-list format
The current allow-list lives in `.kiro/steering/swa-data-scope.md` (pointer) → resolves to
`.kiro/knowledge/swa-stack/allow-list.yml`.

```yaml
confluence:
  spaces:
    - key: CALM
      reason: CALM platform specs and runbooks
    - key: CREWOPS
      reason: Crew-Ops Tech runbooks (FAA Part 117, scheduling)
jira:
  projects:
    - key: CALM
      reason: Pilot tickets
    - key: CREW
      reason: Crew-Member bounded context
figma:
  workspaces:
    - id: <swa-design-workspace-id>
      reason: Design tokens, component frames
gitlab:
  projects:
    - path: swa/calm-platform
      access: feature-branch-write, mr-create-only, no-protected-branch
    - path: swa/crew-member
      access: read-only
```

## Enforcement

1. Every agent that calls a tool in this list MUST first verify the resource id against the allow-list.
2. The `data-scope-guard` hook (`.kiro/hooks/agent-turn/data-scope-guard.json`) intercepts every external tool call.
3. Out-of-scope id → tool call refused, AIQ record emitted with `guardrails.injection_blocks++`, surfaced to user.
4. Activity logged to `.kiro/validaite/telemetry/<agent>.jsonl`.
5. Write-back operations (Jira post, Figma comment) always go through HITL.

## Audit

Per-sprint report aggregates:
- Total tool calls per scope
- Refused / approved counts
- Override events (if any)
- New scopes proposed and the date SWA Cyber approved them.
