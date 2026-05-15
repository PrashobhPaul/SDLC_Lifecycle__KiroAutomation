# Review — Metrics (Code Reviewer Output)

```json
{
  "review_id": "<id>",
  "pr": "<url>",
  "ts_errors": 0,
  "eslint_warnings": 0,
  "coverage_overall": 0.84,
  "coverage_per_layer": {
    "domain": 0.97,
    "application": 0.92,
    "adapters": 0.83,
    "handlers": 0.71,
    "components": 0.74
  },
  "findings": {
    "critical": 0,
    "major": 3,
    "minor": 9
  },
  "graphql": {
    "breaking_changes_flagged": 0,
    "n_plus_one_risks": 1
  },
  "iac": {
    "tfsec_high_or_critical": 0,
    "tflint_errors": 0,
    "checkov_failed": 0,
    "destroy_in_plan": false
  },
  "false_positive_rate_calibration": null,
  "verdict": "approve-with-comments"
}
```
