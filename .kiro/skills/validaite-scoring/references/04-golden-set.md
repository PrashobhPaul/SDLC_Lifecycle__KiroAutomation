# Golden-Set Comparison

## Purpose
Detect regressions between sprints. Same input → expect comparable AIQ on the same artifact-class.

## Layout
```
.kiro/validaite/golden-runs/
├── S1/
│   ├── manifest.json        # what was run, with which model, prompt version, artifact list
│   ├── inputs/              # frozen input set (NL, Jira refs, Confluence refs, Figma refs)
│   ├── outputs/             # produced artifacts
│   └── scorecards/          # AIQ per artifact + composite
├── S2/
│   └── ...
```

## Manifest schema
```json
{
  "sprint": "S<n>",
  "ts": "<RFC3339>",
  "model": { "feature-builder": "...", "developer": "...", ... },
  "judge_prompt_version": "v1.2.0",
  "agent_versions": { "feature-builder": "git-sha", ... },
  "skill_checksums": { "<skill>": "<sha256>", ... },
  "input_seeds": ["seed-1", "seed-2", ...],
  "artifact_count": 20
}
```

## Comparison job
```bash
node tools/golden-compare.ts --base S<n-1> --head S<n>
```
Outputs:
- Per-artifact AIQ delta
- Per-axis median delta
- New regressions (artifacts where the head AIQ dropped >= 0.5)
- Improvements (head − base >= 0.5)
- A summary written to `.kiro/validaite/scorecards/golden-S<n-1>-vs-S<n>.md`

## Phase 1 acceptance
- No artifact class drops more than 0.3 between sprints.
- Composite AIQ trend non-decreasing across S1 → S6.
- New capability (e.g., VTL added in S3) does not degrade pre-existing capability AIQ on the golden set.

## Determinism
- Golden runs use temperature=0 where the model supports it.
- Non-deterministic outputs (e.g., free-form prose) are excluded from the AIQ trend; tracked separately.
