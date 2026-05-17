# AWS CLI

**TL;DR** — Command-line interface for every AWS API. Install once, configure profiles, automate everything. The Swiss Army knife of AWS dev.

## Install (macOS)

```bash
brew install awscli
aws --version
# aws-cli/2.x.x ...
```

Other platforms: see https://aws.amazon.com/cli/

## Configure

### One-time with access keys (not recommended for humans)

```bash
aws configure
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region name: ap-south-1
# Default output format: json
```

Creates `~/.aws/credentials` and `~/.aws/config`.

### SSO via Identity Center (preferred for humans)

```bash
aws configure sso --profile dev
# Step through browser flow; configures a short-lived session
```

`~/.aws/config`:
```ini
[profile dev]
sso_session = mycompany
sso_account_id = 123456789012
sso_role_name = DeveloperAccess
region = ap-south-1
```

Then:
```bash
aws sso login --profile dev
aws s3 ls --profile dev
```

### EC2/Lambda/ECS — no config needed

The instance/Lambda/task IAM role is auto-picked. Just call commands.

## Common patterns

### Switch region / profile per command

```bash
aws s3 ls --profile staging --region us-east-1
```

### Set defaults in env vars

```bash
export AWS_PROFILE=dev AWS_REGION=ap-south-1
aws s3 ls
```

### Query/filter output (JMESPath)

```bash
aws ec2 describe-instances \
  --query 'Reservations[].Instances[?State.Name==`running`].[InstanceId,InstanceType,Tags[?Key==`Name`].Value | [0]]' \
  --output table
```

### Output formats

`--output json` (default) / `text` / `table` / `yaml`.

### Pagination

```bash
aws s3api list-objects-v2 --bucket my-bucket --max-items 10 --starting-token <token>
# Or auto-paginate:
aws s3api list-objects-v2 --bucket my-bucket --no-paginate   # disables
```

Most `list-*` / `describe-*` commands paginate automatically when piped.

### Dry-run (some commands)

```bash
aws ec2 run-instances --image-id ami-xxx --dry-run
```

### Wait commands

Block until a state is reached:
```bash
aws ec2 wait instance-running --instance-ids i-xxx
aws cloudformation wait stack-create-complete --stack-name foo
```

## Useful commands cheat sheet

| Task | Command |
|---|---|
| Current identity | `aws sts get-caller-identity` |
| List EC2 instances | `aws ec2 describe-instances` |
| List S3 buckets | `aws s3 ls` |
| Copy to S3 | `aws s3 cp file s3://bucket/key` |
| Sync folder | `aws s3 sync ./dist s3://bucket/dist --delete` |
| Run Lambda | `aws lambda invoke --function-name fn --payload '{}' out.json` |
| Tail CloudWatch log | `aws logs tail /aws/lambda/fn --follow` |
| SSM session | `aws ssm start-session --target i-xxx` |
| List secrets | `aws secretsmanager list-secrets` |
| Decode error responses | `aws sts decode-authorization-message --encoded-message <msg>` |

## Aliases (`~/.aws/cli/alias`)

```ini
[toplevel]
whoami = sts get-caller-identity
running-ec2 = ec2 describe-instances --query "Reservations[].Instances[?State.Name=='running'].InstanceId" --output text
```

Then: `aws whoami`.

## Pricing

- **CLI itself is free.** You pay for API calls' downstream effects (no metering on the request itself for most services).

## Gotchas

- **Always check `--profile` and `--region`** before running destructive commands. Set up your shell prompt to show them.
- **`aws s3 sync --delete`** can wipe a bucket if pointed wrong. Triple-check.
- **Output format is global default**, can change per command.
- **Auto-pagination** has limits per command — `--max-items` controls it.
- **v1 of AWS CLI is unsupported** — use v2.

## Related

- [SDKs](./sdks.md)
- [CloudShell](./cloudshell.md)
- [IAM](../05-security-iam/iam.md)
