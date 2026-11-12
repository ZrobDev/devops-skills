# GitHub to AWS with OIDC

No access key is stored. The job asks GitHub for an identity token; AWS STS exchanges it for
short-lived credentials of a role whose trust policy names exactly that job.

## One-time, in AWS

1. Identity provider: `token.actions.githubusercontent.com`, audience `sts.amazonaws.com`.
2. One role per purpose (a read-only `plan` role, a `deploy` role). Trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com" },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
        "token.actions.githubusercontent.com:sub": "repo:OWNER/REPO:environment:production"
      }
    }
  }]
}
```

`sub` shapes: `repo:OWNER/REPO:environment:NAME` for a job with `environment:`;
`repo:OWNER/REPO:ref:refs/heads/main` for a branch; `repo:OWNER/REPO:pull_request` for PR
jobs. Use `StringEquals`, never `StringLike` with a trailing `*` on the repo.

3. Permissions policy: least privilege for the resources Terraform manages, plus the state
   bucket and lock table. The plan role gets read-only on the same.

## In the workflow

```yaml
permissions:
  id-token: write
  contents: read
steps:
  - uses: aws-actions/configure-aws-credentials@<sha> # v6.x
    with:
      role-to-assume: ${{ vars.AWS_DEPLOY_ROLE_ARN }}
      role-session-name: gha-${{ github.run_id }}
      aws-region: us-east-1
```

The role ARN is an identifier, not a secret: a repository variable is the right place.

## Terraform state

Never commit `*.tfstate*`. Create the state bucket (versioned, public access blocked) and a
DynamoDB lock table by hand once; configure the `s3` backend; `.gitignore` the state and
`.terraform/`. A plan on a PR uses the read-only role and posts the plan text as a comment,
passed through a file and `env`, never interpolated.

## Before the first apply

Billing alarm on the account. Budget the showcase at cents per month and alarm at one dollar.
