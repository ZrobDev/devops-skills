---
name: release
description: Use when cutting a release of a repo this owner maintains - a skills plugin, a reusable-workflow repo, a tool - "tag a release", "ship v0.2.0", "move the v1 tag", "publish release notes". Checks preconditions, bumps the manifest where one exists, signs and pushes the tag, creates the GitHub release, and moves a floating major tag with a pause before the force push.
license: MIT
allowed-tools: Read, Grep, Bash(git tag:*), Bash(git push:*), Bash(git fetch:*), Bash(git status:*), Bash(git log:*), Bash(git ls-remote:*), Bash(git rev-parse:*), Bash(git branch:*), Bash(gh release:*), Bash(gh run:*), Bash(gh pr:*), Bash(gh api:*)
---

# Release

A release is a tag the owner signed on a commit CI already proved. Nothing here builds or
tests; if that is missing, stop and say so.

## Preconditions (all must hold)

1. On `main`, clean tree, `git fetch origin` done, `HEAD` equals `origin/main`.
2. The newest run on `HEAD` is green: `gh run list --commit $(git rev-parse HEAD) --json conclusion,workflowName`.
   Nothing pending, nothing red, nothing skipped that normally runs.
3. `CHANGELOG.md` has a `## [x.y.z] - YYYY-MM-DD` section for this version at the top.
4. Where a manifest carries a version (`.claude-plugin/marketplace.json`, `pyproject.toml`,
   `package.json`), it already says `x.y.z` and the repo's validator passes.
5. The version is SemVer: breaking input or output changes bump the major, additive
   changes the minor, fixes the patch. A reusable workflow that adds an input is minor;
   one that renames or removes an input is major and gets a new floating tag (`v2`).

Items 3 and 4 are changes to `main`; if they are missing, they go through a PR first.

## Procedure

1. `git tag -s vX.Y.Z -m vX.Y.Z` then `git push origin vX.Y.Z`. Tags are signed with the
   same key as commits. Immutable: fix forward with a new tag, never delete or move one.
2. `gh release create vX.Y.Z --verify-tag --generate-notes`. Read the generated notes; if a
   CHANGELOG section exists, prefer `--notes-file` with that section.
3. **Reusable-workflow repos only** (callers use `@v1`): move the floating major.
   Say what will happen, wait for an explicit yes, then
   `git tag -f -s v1 vX.Y.Z -m v1 && git push -f origin v1`. Every caller picks the new
   code up on its next run, so this is the one step that can break many repos at once. The
   repo's tag ruleset must leave `v1` unprotected while protecting `v*.*.*`.
4. Verify: `git ls-remote --tags origin` shows `vX.Y.Z` (and `v1`) at the expected commit;
   `gh release view vX.Y.Z` exists.

## Handing back

Report the tag, the commit, the release URL, and for a moved floating tag the before and
after commits. One line each. If a precondition failed, report which and stop.

Observed failures: `references/gotchas.md`.
