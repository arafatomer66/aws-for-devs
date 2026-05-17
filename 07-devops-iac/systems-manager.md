# Systems Manager (SSM)

**TL;DR** — Big umbrella service. Notable pieces: **Session Manager** (no-SSH shell into EC2), **Patch Manager**, **Run Command** (run scripts on fleets), **Parameter Store**, **State Manager**, **Inventory**, **Automation runbooks**, **OpsCenter**.

## The most-used pieces

### Session Manager
Connect to EC2 (or any SSM-managed instance, including on-prem) **without SSH**, via IAM, all logged. Replaces bastion hosts.

```bash
aws ssm start-session --target i-0abcdef1234567890
```

Prereqs: SSM Agent installed (default on AL2023/Ubuntu AMIs) and an IAM instance role with `AmazonSSMManagedInstanceCore`.

### Run Command
Run a script/command across a fleet:

```bash
aws ssm send-command --document-name "AWS-RunShellScript" \
  --targets "Key=tag:Env,Values=prod" \
  --parameters 'commands=["uptime", "df -h"]'

aws ssm get-command-invocation --command-id <id> --instance-id i-xxx
```

### Patch Manager
Define a patch baseline + a maintenance window; SSM patches your fleet on schedule.

### Parameter Store
[Already covered](../05-security-iam/parameter-store.md).

### State Manager
Continuously enforce desired state (e.g., ensure CloudWatch agent installed, file permissions correct).

### Automation
Runbooks (YAML) — multi-step workflows for ops (restart RDS, rotate AMIs, snapshot fleet).

### Inventory
Periodically collect installed packages, configs, etc. — view as a queryable inventory.

### OpsCenter / Incident Manager
Issue/incident tracking aware of AWS resources; can auto-trigger Automation runbooks.

## Real-world example

> Fleet of 30 EC2s:
> - SSM Agent on each (managed instances).
> - Engineers use Session Manager — no SSH ports open, all sessions logged to CloudWatch.
> - Patch Manager keeps OS patched weekly.
> - State Manager keeps CloudWatch Agent installed + config in sync.
> - Run Command rolls out emergency fixes in 30 seconds.

## Pricing

- Most SSM features are **free**.
- Session Manager: data transfer charges only.
- Automation: free for AWS-owned runbooks.
- Advanced parameters / Parameter Store advanced tier: $0.05/param/mo.
- OpsCenter: small per-incident fee.

## Gotchas

- **SSM Agent + IAM role on instance are required.** Without them, nothing works.
- **VPC endpoint for SSM** is needed for private-subnet instances (3 endpoints: ssm, ec2messages, ssmmessages).
- **Session logs** go to S3/CloudWatch — turn on for audit.
- **Patch baselines** — read carefully; severity filters and approval delays matter.

## Related

- [Parameter Store](../05-security-iam/parameter-store.md)
- [CloudWatch](../08-monitoring-observability/cloudwatch.md)
- [EC2](../01-compute/ec2.md)
