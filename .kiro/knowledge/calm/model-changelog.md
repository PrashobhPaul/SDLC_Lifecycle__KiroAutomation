# Model Changelog

Records every model change applied to a CALM agent and the calibration impact.

| Date       | Agent           | From                  | To                      | Calibration re-run | Notes                              |
|------------|-----------------|-----------------------|-------------------------|--------------------|------------------------------------|
| 2026-04-15 | code-reviewer   | claude-sonnet-4-6     | claude-opus-4-6         | YES (S2)           | Higher recall on CRITICAL findings |
| 2026-04-15 | developer       | claude-sonnet-4-6     | claude-opus-4-6         | YES (S2)           | Plan stability; SDK v3 compliance  |

## Rule

Any model change requires a calibration re-run **before** scorecards from the new
model are accepted into the sprint roll-up. Until calibration completes, AIQ
records from the new model are tagged `provisional=true` and excluded from the
sprint average.
