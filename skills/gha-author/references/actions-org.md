# Official actions, by job

The `actions` organisation on GitHub publishes the actions below. Pin each to a 40-char SHA
with the version in a comment; Dependabot moves the pin. Majors current as of 2026-11.

| Need | Action | Notes |
|---|---|---|
| Get the code | `actions/checkout` v7 | `persist-credentials: false`; `fetch-depth: 0` only for history scans |
| Python | `astral-sh/setup-uv` v10 (third party) over `actions/setup-python` v6 | `enable-cache: true`, `python-version` input; `uv sync --frozen` |
| Node | `actions/setup-node` v6 | `cache: pnpm` or `npm`; never `npx` of an unpinned package |
| Go, Java, .NET | `actions/setup-go` v6, `setup-java` v5, `setup-dotnet` v5 | built-in caches |
| Cache anything else | `actions/cache` v5 | key from a lockfile hash, never from PR input |
| Pass files between jobs | `actions/upload-artifact` v5, `download-artifact` v6 | short `retention-days` |
| Script the API | `actions/github-script` v9 | pass data via `env`, read `process.env` in the script |
| Vulnerable deps on a PR | `actions/dependency-review-action` v4 | public repos, or Advanced Security |
| Build provenance | `actions/attest-build-provenance` v3 | needs `id-token: write`, `attestations: write` |
| Static site | `actions/configure-pages`, `upload-pages-artifact`, `deploy-pages` | the only sanctioned Pages path |
| Label PRs by path | `actions/labeler` v6 | `pull_request_target` by design; give it no secrets |
| Stale issues | `actions/stale` v10 | a schedule, so it spends minutes monthly |
| App token instead of PAT | `actions/create-github-app-token` v2 | scoped, short-lived |
| Starter YAML | `actions/starter-workflows` | read for ideas; never copy its unpinned `uses:` |

Outside the organisation but part of the same toolbelt: `aws-actions/configure-aws-credentials`
(OIDC), `hashicorp/setup-terraform`, `tailscale/github-action` (OIDC federated identity),
`tailscale/gitops-acl-action`.

Runner facts that matter: `ubuntu-latest` already has `gh`, `jq`, `shellcheck`, `pipx`,
`docker`; it does not have `uv`, `actionlint` or `zizmor`.
