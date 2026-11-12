---
name: deploy-vps
description: Use when wiring a repository to deploy to a self-managed host over the tailnet from GitHub Actions - "add CD for this repo", "deploy to the VPS from CI", "run the apply script after the gate passes". Produces the dispatch-triggered caller, the host-side user and sudoers rule, the tailnet policy change, and the first-run checks. No secret is stored in GitHub.
license: MIT
allowed-tools: Read, Grep, Bash(gh workflow:*), Bash(gh run:*), Bash(gh variable:*), Bash(gh api:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*)
---

# Deploy to a tailnet host from Actions

The shape, from the runner inward: an OIDC token proves the job's identity; a Tailscale
OAuth client of type "OIDC federated identity" turns it into an ephemeral `tag:ci` node;
Tailscale SSH on the host trusts that node as one Unix user; sudoers lets that user run
one script. Each layer is reviewed in its own repo. The reusable workflow does none of the
deciding; it only joins and runs the command.

## Before touching anything

- Confirm the plan: on a private Free-plan repo the trigger is `workflow_dispatch` only
  and the `environment:` is a namespace, not a gate (see `gha-author`'s
  `references/free-plan-limits.md`).
- Confirm the apply script is non-interactive end to end: run its `--dry-run` over a plain
  SSH session with no terminal (`ssh -T`). Any prompt, any need for a user session
  variable that is missing without a login shell, stops the work here.
- Confirm the host checkout is clean. The deploy script fast-forwards only.

## Procedure

1. **Tailnet policy** (its own repo, applied by CI): `tagOwners` for `tag:ci`; a grant from
   `tag:ci` to the host on tcp/22; an `ssh` rule `tag:ci` to the host as user `deploy`.
   One host, one user.
2. **OAuth client** (admin console): type OIDC federated identity, issuer
   `token.actions.githubusercontent.com` (https), subject
   `repo:OWNER/REPO:environment:production`, a fixed audience string, scope `auth_keys`,
   tag `tag:ci`. Record client id and audience as repository variables:
   `gh variable set TS_OAUTH_CLIENT_ID` and `gh variable set TS_OIDC_AUDIENCE`.
3. **Host** (root, by the operator, each step confirmed): `tailscale set --ssh`; create a
   passwordless `deploy` user; a sudoers drop-in letting `deploy` run exactly
   `/srv/projects/REPO/vps/deploy-from-ci.sh` as the service user; `visudo -c`.
4. **Deploy script in the repo**, `vps/deploy-from-ci.sh`: export `XDG_RUNTIME_DIR` and
   `DBUS_SESSION_BUS_ADDRESS` for the service user (sudo gives no session); fetch; exit 0
   with `up-to-date` when a marker file holds `origin/main` already; else `merge --ff-only`,
   run the existing apply script, and write the marker only after it succeeds. Reviewed
   like any other change.
5. **Caller**: copy the pipeline repo's `templates/deploy-caller.yml` to
   `.github/workflows/deploy.yml`; set host, user, command; keep `workflow_dispatch` as the
   only trigger on private repos. Run `gha-author`'s lint step.
6. **Environment**: `gh api -X PUT repos/OWNER/REPO/environments/production` with no
   protection rules on Free. Variables, not secrets: there are none to store.

## First run and proof

1. From a tailnet laptop allowed by a temporary `ssh` rule:
   `ssh deploy@HOST sudo -n -u SERVICE_USER /srv/projects/REPO/vps/deploy-from-ci.sh --dry-run`
   exits 0 with no prompt. Remove the temporary rule.
2. `gh workflow run deploy.yml`, then `gh run watch`. The `tag:ci` node appears in the
   admin console during the run and is gone after it.
3. Remote `HEAD` equals `origin/main`.
4. A second dispatch prints `up-to-date` and restarts nothing.
5. Negative check: `ssh deploy@HOST sudo -n -u SERVICE_USER bash` is refused.

Report the run URL, the before and after commit on the host, and the five checks.

Observed failures: `references/gotchas.md`.
