# Lessons from the classic "Intro to CI/CD" sample

The widely forked sample (an Express app, Terraform for EC2, deploy workflows) is from
2021 to 2023. What still holds, and what to replace.

Keep:
- CI on `pull_request` with `paths:` filters; one job per concern (test, lint).
- Staging before production, expressed as two `environment:` jobs with `needs:`.
- Environments with required reviewers as the human gate (public repos or paid plans).
- OIDC to the cloud (`aws-actions/configure-aws-credentials` with `role-to-assume`),
  `permissions: id-token: write`. No stored cloud keys.
- Terraform plan posted to the PR as a collapsed comment; apply only after the plan passed.
- `workflow_dispatch` with a validated input (the sample checks the PR number against the API
  before using it).
- CodeQL on public repos.

Replace:
- `actions/checkout@v2`, `github-script@v5`, `::set-output` (removed; use `$GITHUB_OUTPUT`).
- Tag pins (`@v2`) with SHA pins and a version comment.
- `rsync --delete` plus `ssh` with a stored private key: use an identity the platform issues
  per run (OIDC federated identity for Tailscale, OIDC for clouds) and a host-side forced
  command or sudoers rule.
- `action-tmate` on failure: it opens a shell into the runner for anyone with the link.
- `first-interaction` bots and `brew install` inside a job: minutes spent for nothing.
- Committing generated Terraform files back from the workflow with the job token.
