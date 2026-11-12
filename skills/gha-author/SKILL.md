---
name: gha-author
description: Use when writing, fixing or reviewing a GitHub Actions workflow or a reusable workflow caller - adding CI to a repo, pinning actions, tightening permissions, wiring a deploy or an OIDC cloud login, or when zizmor or actionlint is red. Produces least-privilege, SHA-pinned, minutes-aware workflows and runs the linters before handing back.
license: MIT
---

# GitHub Actions authoring

Start from something that already runs. Writing a job from scratch is the last option.

## Procedure

1. **Reuse first.** Look for a reusable workflow or caller template the owner already
   maintains (a `devops-pipeline`-style repo with `security.yml`, `python-ci.yml`,
   `deploy-*.yml` and `templates/`). A caller is ten lines; copy it and change the inputs.
2. **Decide the trigger from the budget.** Tests and lint run on `pull_request` only.
   `push: main` repeats what the PR proved and spends the same minutes twice. Deploys run
   on `workflow_dispatch` unless the repo can protect its environment (see
   `references/free-plan-limits.md`). Never `pull_request_target` or `issue_comment`
   without a written reason and an owner-only `if:`.
3. **Write the job** against the checklist below. Every `uses:` gets resolved to a 40-char
   SHA with the version as a comment; `pinact run` does it in place, or
   `gh api repos/OWNER/REPO/git/ref/tags/vX.Y.Z` (dereference annotated tags with
   `git/tags/<sha>`). Choose releases at least two weeks old.
4. **Lint, then fix, then lint again:** `actionlint` and
   `zizmor --min-severity medium .github/workflows`. Treat both as blocking. Suppress a
   zizmor finding only with the rule's documented config and a comment saying why.
5. **Prove it can fail.** A new test job gets one scratch commit with a failing assert;
   a new gate gets one fixture it must flag. Watch the run go red, then revert.
6. If `ci-reviewer` exists in this plugin, hand it the diff of `.github/` and resolve its
   high findings before opening the PR.

## Checklist (every workflow)

- `permissions: contents: read` at the top, or `permissions: {}` when jobs set their own.
  A job escalates only the scope one of its steps uses (`id-token: write` for OIDC,
  `pull-requests: write` for a comment). Nothing gets `write-all`.
- `actions/checkout` with `persist-credentials: false` unless a later step pushes.
- `timeout-minutes` on every job. `concurrency` keyed on the PR number for CI
  (`cancel-in-progress: true`) and on the environment for deploys (`false`).
- Inputs, outputs, event payload text and secrets reach `run:` only through `env:`.
  A `${{ }}` inside `run:` is template injection, and zizmor fails it.
- Reusable workflows: inputs typed with defaults; `secrets:` declared; the caller passes
  `secrets: inherit` only to the owner's own reusable workflows.
- Third-party binaries: release tarball by version plus `sha256sum -c`, never a
  download piped straight into a shell. Python tools by exact version through `pipx` or `uv tool install`.
- One deploy job per environment; the environment name appears exactly once and matches
  the OIDC `sub` any cloud or tailnet trust policy expects.
- Cloud or tailnet login is OIDC. A stored long-lived key needs a written reason.
- Dependabot `github-actions` ecosystem with a cooldown, grouped, capped PR count.

## Handing back

Report: trigger and why; permissions per job; which pins were resolved and to what;
the actionlint and zizmor outcome; the break-the-guard evidence; estimated minutes per
run. Never say "CI is green" without a run URL.

## References

- `references/actions-org.md`: which official `actions/*` to use for what.
- `references/oidc-aws.md`: trust policy for GitHub to AWS with no stored keys.
- `references/free-plan-limits.md`: what a Free plan cannot do, and the minutes query.
- `references/ci-cd-intro-lessons.md`: what the classic intro tutorial still teaches,
  and what to replace.
- `references/gotchas.md`: observed failures.
