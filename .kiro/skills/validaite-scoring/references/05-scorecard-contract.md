# Sprint Scorecard Contract

## What the aggregator produces per sprint

```
.kiro/validaite/scorecards/S<n>/
├── chain-fidelity.json      # FB→Dev, Dev→Test, Test→Reviewer, Reviewer→Doc handshake rates
├── governance.json          # G1–G9 fire counts, override expiry, allow-list adherence
├── per-agent/
│   ├── feature-builder.json
│   ├── developer.json
│   ├── test-engineer.json
│   ├── code-reviewer.json
│   └── documentation.json
├── per-layer/               # FE / GQL / BE / VTL / IaC / DOCS / QA roll-up
├── golden-vs-prev.md        # Δ vs prior sprint
├── calibration.md           # if Sprint 2 or 5
├── README.md                # human-readable narrative for SWA stakeholders
└── prashant-self-serve.md   # how to reproduce + interpret
```

## Required fields per per-agent JSON

```json
{
  "agent": "<name>",
  "sprint": "S<n>",
  "turns": 0,
  "tokens_in": 0,
  "tokens_out": 0,
  "latency_ms_p50": 0,
  "latency_ms_p95": 0,
  "hitl": {
    "gates_required": 0,
    "gates_fired": 0,
    "compliance_pct": 100.0,
    "approvals": 0,
    "rejections": 0,
    "modifications": 0
  },
  "guardrails": {
    "input_pass_rate": 1.0,
    "output_pass_rate": 1.0,
    "pii_redactions": 0,
    "injection_blocks": 0,
    "schema_validation_errors": 0
  },
  "verification": {
    "tsc_pass_rate": 1.0,
    "eslint_pass_rate": 1.0,
    "graphql_codegen_pass_rate": 1.0,
    "tf_validate_pass_rate": 1.0,
    "tfsec_pass_rate": 1.0,
    "test_pass_rate": 1.0,
    "coverage_pct_avg": 80.0
  },
  "aiq": {
    "composite_avg": 4.0,
    "axes": { "pattern_adherence": 4.0, "correctness": 4.0, "completeness": 4.0, "security": 4.0, "maintainability": 4.0, "handshake_fidelity": 4.0 }
  },
  "phase1_targets": {
    "compilation_pass_rate": 0.98,
    "pattern_adherence_min": 4.0,
    "hallucination_rate": 0.02,
    "handshake_fidelity": 0.99,
    "first_pass_acceptance": 0.6,
    "reviewer_fp_rate": 0.15,
    "token_reduction_vs_v1_pct": 30.0
  }
}
```

## Self-serve narrative
The `prashant-self-serve.md` is the file SWA's tech lead opens to interpret the sprint. It contains:
- One-line summary
- The 10 Phase-1 metrics with current values vs target
- The 3 most interesting traces (one good, one bad, one boundary)
- The next-sprint risks raised by this sprint's data
- The exact command to reproduce: `node tools/validaite-aggregate.ts --sprint S<n> --explain`
