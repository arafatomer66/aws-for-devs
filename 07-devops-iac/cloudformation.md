# CloudFormation

**TL;DR** — AWS's native Infrastructure-as-Code. YAML/JSON templates → stacks. Free. Slow but reliable. Most CDK/SAM users still use CFN under the hood.

## What it is

A declarative IaC service. You write a template describing AWS resources (and their dependencies); CloudFormation creates/updates/deletes them as one **stack** with rollback on failure.

## Key concepts

- **Template** — YAML/JSON file declaring resources.
- **Stack** — deployed instance of a template.
- **Change Set** — preview of what will change before applying.
- **Drift detection** — find resources that diverge from the template.
- **Stack Set** — deploy the same template across many accounts/regions.
- **Nested stacks** — a stack that's a resource within another stack.
- **Custom resource** — Lambda-backed resource for things CFN doesn't natively support.
- **Cross-stack reference** — `Outputs` from one stack `Fn::ImportValue` in another.

## Template structure

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "ShareDeal storage stack"

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

Mappings:
  EnvConfig:
    dev:     { RetentionDays: 7 }
    prod:    { RetentionDays: 90 }

Resources:
  UploadsBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "selefe-uploads-${Environment}"
      VersioningConfiguration: { Status: Enabled }
      LifecycleConfiguration:
        Rules:
          - Id: ExpireOld
            Status: Enabled
            ExpirationInDays: !FindInMap [EnvConfig, !Ref Environment, RetentionDays]

  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref UploadsBucket
      PolicyDocument:
        Statement:
          - Effect: Deny
            Principal: "*"
            Action: s3:*
            Resource: !Sub "${UploadsBucket.Arn}/*"
            Condition: { Bool: { "aws:SecureTransport": false } }

Outputs:
  BucketName:
    Value: !Ref UploadsBucket
    Export: { Name: !Sub "${AWS::StackName}-BucketName" }
```

## Usage

```bash
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name selefe-storage-dev \
  --parameter-overrides Environment=dev \
  --capabilities CAPABILITY_NAMED_IAM

aws cloudformation describe-stacks --stack-name selefe-storage-dev
aws cloudformation delete-stack --stack-name selefe-storage-dev
```

### Change set preview

```bash
aws cloudformation create-change-set \
  --stack-name selefe-storage-dev \
  --template-body file://template.yaml \
  --change-set-name v2

aws cloudformation describe-change-set --stack-name selefe-storage-dev --change-set-name v2
```

## CloudFormation vs CDK vs Terraform

| | CloudFormation | CDK | Terraform |
|---|---|---|---|
| Language | YAML/JSON | TS/Py/Java/Go/.NET | HCL |
| State | AWS-managed | AWS-managed (synth → CFN) | tfstate (S3 + DDB lock) |
| Multi-cloud | No | No | Yes |
| Loops/conditions | Awkward | Real code | Functions + counts |
| Speed | Slow | Slow (uses CFN) | Faster |
| Ecosystem | Smaller | Growing | Largest |

**Most teams using AWS only**: CDK (writes CFN under the hood).
**Multi-cloud or heavy state mgmt**: Terraform.

## Real-world example

> Multi-environment stack:
> - Same template deployed as 3 stacks: `web-dev`, `web-staging`, `web-prod`.
> - Parameters differ (`InstanceType`, `MinCapacity`).
> - StackSet deploys across `us-east-1` + `ap-south-1` for DR.

## Pricing

- **Free.** You pay for the resources created.

## Gotchas

- **Slow.** A stack with 20 resources takes minutes. EKS/RDS stacks can be 30 min.
- **Rollback on failure** is good but can leave intermediate state if rollback also fails.
- **Updates require replacement** for some resources (e.g., changing a bucket name → recreates → deletes data). Read the docs per property.
- **Deletion protection** is per-stack — turn on for prod.
- **Cyclic dependencies** are common; use Outputs/Exports carefully.
- **Drift detection is opt-in** — drift happens (someone clicked in console).
- **Stack quotas:** 500 resources per stack default — split into nested stacks.

## Related

- [CDK](./cdk.md) — preferred for new work
- [SAM](./sam.md) — CFN for serverless
- [Terraform](#) — non-AWS, popular alternative
