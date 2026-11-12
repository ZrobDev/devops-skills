# devops-skills

Lean skills for GitHub Actions and deploys, for Claude Code and any skills.sh-compatible
agent. Each one encodes a working pipeline the owner runs, not a tutorial.

| Skill | Use it for |
|---|---|
| `gha-author` | Write or fix a workflow: SHA pins, least privilege, `pull_request`-only tests, OIDC logins, actionlint and zizmor before handing back |
| `ci-triage` | A red run: read the failing log, classify (security gate, our bug, flake, infra incl. the minutes cap), propose the fix, never re-run or merge |
| `release` | Signed SemVer tags, GitHub releases, and moving a reusable-workflow repo's floating `v1` with a pause |
| `deploy-vps` | Deploy to a tailnet host from Actions over Tailscale SSH with OIDC federated identity: no secret stored in GitHub |

References worth reading on their own, under `skills/gha-author/references/`: the official
`actions/*` catalogue by job, GitHub to AWS with OIDC, what a Free plan cannot do (and the
minutes query), and what the classic "Intro to CI/CD" sample still teaches.

## Install

Claude Code, as a plugin:

```
/plugin marketplace add ZrobDev/devops-skills
/plugin install devops@zrobdev-devops
```

Any agent, through skills.sh (pin the CLI version; never run an unpinned package):

```
pnpm dlx skills@<pinned-version> add ZrobDev/devops-skills
```

Installs are cached at the version they were installed. `claude plugin update devops@zrobdev-devops`
after a release.

## What these skills assume

- The owner keeps reusable workflows in a public `devops-pipeline`-style repo
  (`security.yml`, `python-ci.yml`, `deploy-tailscale-ssh.yml`) and callers pin them `@v1`.
  Without such a repo, `gha-author` still applies; `deploy-vps` needs the reusable workflow.
- Tests run on `pull_request` only. Actions minutes on a Free plan are one shared budget.
- Deploy approval on a private Free-plan repo is the human running `workflow_dispatch`.

## How this repo changes

A skill changes after an observed failure, not an idea. Open a "Skill feedback" issue with
the evidence; the fix lands in `references/gotchas.md` first. See [AGENTS.md](AGENTS.md)
and [CONTRIBUTING.md](CONTRIBUTING.md).

## Security

Plain text only, no scripts to run, URLs limited to an allowlist, every action and tool
pinned, CI from the owner's public security gate plus a skill scanner. Details and how to
report a problem: [SECURITY.md](SECURITY.md).
