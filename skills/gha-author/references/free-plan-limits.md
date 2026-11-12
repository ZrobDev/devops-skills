# What a Free plan does not give a private repo

Verified against the API on 2026-11-12.

| Feature | Private repo on Free | Public repo on Free |
|---|---|---|
| Environment protection rules (required reviewers, wait timer, branch filter) | no (HTTP 422) | yes |
| Rulesets, branch protection | no (HTTP 403) | yes |
| Secret scanning, push protection | partial | yes |
| Dependency review action, CodeQL | no | yes |
| Actions minutes | 2,000 per month, shared across all private repos | unlimited for public |

Consequences:

- An `environment:` on a private repo scopes secrets and variables and labels the
  deployment, but any workflow on any branch can reference it. Treat it as a namespace, not
  a gate. The approval is the human running `workflow_dispatch`.
- A deploy that needs a real approval gate lives in a public repo, or the plan is upgraded.
- The month's minutes are checked with
  `gh api '/users/OWNER/settings/billing/usage?year=YYYY&month=M'` and summing the
  `usageItems` with `product` equal to `actions`. The older
  `/settings/billing/actions` endpoint returns HTTP 410.
- Cheap habits: `pull_request` only for tests; `timeout-minutes` everywhere; `concurrency`
  with cancel on CI; no scheduled workflows on private repos without a reason.
