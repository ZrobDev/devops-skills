---
name: ci-triage
description: Use when a GitHub Actions run is red and the cause is unknown - "CI failed", "the gate is red", a PR check will not go green, or a run stopped for a billing or quota reason. Reads only logs that already exist, classifies the failure as security gate, our bug, flake or infra, and proposes the fix. Never re-runs, pushes or merges.
license: MIT
allowed-tools: Read, Grep, Bash(gh run:*), Bash(gh pr:*), Bash(gh api:*), Bash(git branch:*), Bash(git log:*)
---

# CI triage

Diagnose why CI is red in the current repo, or the PR or run the user names, and propose
the fix. Zero Actions minutes: everything here reads logs that already exist.

1. **Find the run.** If the user named a run id or PR, use it. Otherwise
   `gh run list --limit 5 --json databaseId,workflowName,headBranch,conclusion,createdAt`
   and take the newest `failure` on the current branch (`git branch --show-current`), else
   the newest failure overall. Say which one you picked.
2. **Read only the failure.** `gh run view <id> --log-failed | tail -200`. If that is
   empty (a reusable-workflow job), `gh run view <id> --json jobs` to find the failed job,
   then `gh run view --job <job-id> --log | tail -200`.
3. **Classify** as exactly one:
   - **security gate**: gitleaks, zizmor, semgrep or trivy. Follow the owning pipeline
     repo's runbook ("When a check is red"). A gitleaks hit is a real secret until you have
     read it: say rotate first. Never propose a `.gitleaksignore` fingerprint or a
     `# nosemgrep` without showing the line and asking.
   - **our bug**: the change broke a test, lint or build. Name the file and line and the fix.
   - **flake**: timing, network, a runner hiccup. Evidence required: the same job passed on
     the same commit, or the error is a known transient. Do not call something a flake to
     make it go away.
   - **infra**: expired secret (for example the Claude review token), a moved action pin, a
     runner image change, or the **minutes cap**. For the cap: the run shows no job logs and
     the Actions page says the account's minutes are exhausted; confirm with
     `gh api '/users/OWNER/settings/billing/usage?year=YYYY&month=M'` and sum the
     `actions` items. The fix is waiting for the reset or cutting a `push` trigger, not a
     code change.
4. **Report**: run link, classification, the 3 to 10 log lines that prove it, and the
   proposed fix as a diff or command.
5. **Stop.** Do not re-run the workflow, push, or edit files until the user says so. Never
   merge the PR.

See `references/gotchas.md` for failures this procedure has misread before.
