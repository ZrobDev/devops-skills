# Gotchas

Observed failures only. Add a line when the skill misfires, with the retro or issue that saw it.

- `gh run view --log-failed` is empty for a job inside a reusable workflow; go through
  `--json jobs` and `--job <id>`.
- A gitleaks "finding" with exit code 1 is a scanner error, not a secret. The pipeline uses
  `--exit-code 42` for findings so the two cannot be confused; check which code the log shows.
- `gh run rerun` reuses the original run's resolved reusable-workflow refs. After moving a
  floating tag such as `v1`, a rerun still runs the old code; push a commit instead (2026-11-12).
