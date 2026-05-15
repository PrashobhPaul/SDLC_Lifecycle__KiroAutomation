# Implementation Plan — Developer Agent Template

## Plan <plan_id>

**Feature:** <feature-id>
**Layers:** react-mfe / graphql / node-resolver / vtl / terraform (delete N/A)
**Estimated effort:** <hours>

## Files to create / modify

| Action | Path                                  | Purpose                       |
|--------|---------------------------------------|-------------------------------|
| CREATE | <path>                                | <one-line purpose>            |
| MODIFY | <path>                                | <one-line purpose>            |

## Implementation steps (ordered)

### Step 1 — <title>
- **What:** <description>
- **Files:** <list>
- **Depends on:** <step ids or `none`>
- **Verification:** <build / test / lint>

### Step 2 — <title>
…

## Integration points

- AppSync schema additions: <none / list>
- Lambda resolver additions: <list>
- Kafka topic interactions: <none / list>
- Terraform module changes: <none / list>
- MFE route changes: <none / list>

## HITL gates that will fire

| Gate | Likely status |
|------|---------------|
| G2   | required      |
| G3   | required if schema breaking |
| G4   | required if Terraform plan has destroy |

## Risks / Assumptions

- <bullet>

## Approval

> Type the literal word `approve` to proceed. Any other response triggers
> revise-plan or reject loop.
