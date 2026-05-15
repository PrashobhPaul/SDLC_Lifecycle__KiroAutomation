# Fallback Modes

## Principle
Every external mutation has a defined "what happens if it can't land". The fallback writes a local
artifact that captures the intent, so the operator can replay or manually complete.

## Per action class

### Jira create / comment
Fallback path: `.kiro/handoff/<feature-id>/jira-pending.md`
Contents: action kind, payload, idempotency key, intended endpoint, why it failed.
On replay: `tools/dlq-replay.ts --kind jira-* --since <ts>` re-enqueues.

### Figma comment
Fallback path: `.kiro/handoff/<feature-id>/figma-pending.md`
Same structure. Lower urgency (informational).

### GitLab MR create
Fallback path: `.kiro/handoff/<feature-id>/mr-pending.md`
Contents include the diff hash and source-branch ref so a manual `glab mr create` is straightforward.
Higher urgency than Figma (blocks pre-merge).

### Inter-agent handoff redrive
Fallback path: `.kiro/handoff/<feature-id>/redrive-pending.json`
Contains the original envelope. Operator can `/agent <to_agent>` and pass the redrive file.

### Notification (Slack / email)
Fallback path: `.kiro/validaite/telemetry/notify-pending.jsonl`
Lowest urgency. Manual flush daily.

## Mode flags
```yaml
# .kiro/validaite/queues/config.yml
fallback:
  jira: write-local
  figma: write-local
  gitlab: write-local
  handoff: write-local
  notify: write-local
  # 'fail-fast' alternative for environments where fallbacks aren't desired (e.g., prod cutover)
```

## Reporting
Sprint scorecard includes `fallback_used_count` per queue and per action class.
