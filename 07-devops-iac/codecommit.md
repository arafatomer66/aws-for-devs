# CodeCommit

**TL;DR** — AWS-managed Git. Effectively in maintenance mode — AWS announced no new customer onboarding in 2024. Use GitHub or GitLab instead.

## What it was

Private Git repos hosted by AWS, IAM-authenticated. Existed mainly to let regulated customers keep code "fully inside AWS."

## Current state (2026)

- **No new accounts can create CodeCommit repos.**
- Existing customers can keep using it.
- AWS now recommends GitHub / GitLab / Bitbucket integration via Source Actions in CodePipeline.

## What to use instead

- **GitHub** — most teams.
- **GitLab** — for self-hosted or GitLab CI lovers.
- **Bitbucket** — Atlassian shops.
- **GitHub via CodePipeline source action** — keeps the AWS-side CI/CD pipeline if you want.

## If you have legacy CodeCommit repos

- Migrate to GitHub via `git remote set-url` + push.
- Update CodePipeline source actions.

## Related

- [CodePipeline](./codepipeline.md)
- [CodeBuild](./codebuild.md)
