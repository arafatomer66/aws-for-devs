# CloudShell

**TL;DR** — Browser-based terminal already authenticated to your AWS account. Pre-installed AWS CLI, kubectl, eksctl, sam, cdk, git, jq, helm. 1 GB persistent home directory. Free.

## What it is

A free, in-console Linux shell. Click the CloudShell icon in the AWS Console → you get a bash prompt with your IAM identity, AWS CLI already configured.

## Why it's handy

- **Quick scripts** without leaving the browser.
- **Recovery** when your laptop is offline.
- **Demo** to show someone a command.
- **No local install** required.

## What's pre-installed

- AWS CLI v2 + Session Manager plugin.
- Git, jq, bash, zsh, fish.
- Python 3, Node.js, npm.
- kubectl, eksctl, helm.
- SAM CLI, CDK.
- Terraform (recent).
- Docker (newer regions).

## Persistent storage

1 GB in `$HOME`, persists across sessions per region. Files outside $HOME are wiped between sessions.

## Pricing

- **Free.**

## Real-world example

> Forgot your laptop on a trip. Need to roll a Lambda back:
> - Open AWS Console → CloudShell.
> - `aws lambda update-function-code --function-name fn --s3-bucket releases --s3-key v122.zip`.
> - Done.

## Gotchas

- **Idle timeout** ~20 min — disconnects, but home dir persists.
- **Per-region** — files in `us-east-1` shell aren't in `ap-south-1` shell.
- **Limited resources** — not a build server.
- **Public IP-restricted SSH** not available; it's only via the browser console.

## Related

- [CLI](./cli.md)
- [Cloud9 (deprecated)](#) — being deprecated in favor of CloudShell + your local IDE
