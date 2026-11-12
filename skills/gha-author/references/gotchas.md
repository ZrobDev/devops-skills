# Gotchas

Observed failures only. Add a line when the skill misfires, with the retro or issue that saw it.

- Required reviewers on an environment returned HTTP 422 "billing plan does not support
  the required reviewers protection rule" on a private Free-plan repo (2026-11-12). Check
  the plan before promising an approval gate.
- A reusable workflow consumed from a public repo must itself be public. Private callers
  of a private reusable are fine; a public caller of a private one fails with
  "workflow was not found".
- `tailscale/github-action` installs the latest client unless `version:` is set, and
  verifies the tarball only when `sha256sum:` is set. Pin both.
- 2026-11-12 dry run of a read-only LLM "ci-reviewer" over 13 repos and 3 templates: zero
  high or medium findings, 8 low (hash-less pipx installs, job-level token visible to an
  install step, a `gh pr comment` wildcard in an AI reviewer's allowed tools, `issues: write`
  likely unneeded, mutable `@v1` receiving a secret, a live semgrep ruleset, missing
  `timeout-minutes` on five jobs, unpinned apt installs). zizmor covered every high-impact
  class, so no agent ships; this list is the checklist's "hygiene" tail instead.
