# SAM — Serverless Application Model

**TL;DR** — CloudFormation shorthand for serverless. Lambda + API Gateway + DynamoDB in a few lines. Local invoke and testing built-in.

## What it is

A CloudFormation transform that adds short serverless-specific resource types (`AWS::Serverless::Function`, `AWS::Serverless::Api`, `AWS::Serverless::HttpApi`, `AWS::Serverless::Table`). The SAM CLI builds, packages, deploys, and runs Lambdas locally.

## Why it exists

CFN for serverless is verbose. SAM compresses 50 lines of "create Lambda + role + permission + API GW + integration" into 10 lines.

## Key concepts

- **Template** — `template.yaml`, with `Transform: AWS::Serverless-2016-10-31`.
- **`AWS::Serverless::Function`** — Lambda + IAM role + event mappings.
- **`AWS::Serverless::HttpApi`** — API Gateway v2.
- **`AWS::Serverless::Api`** — API Gateway v1.
- **`AWS::Serverless::Table`** — DynamoDB.
- **`AWS::Serverless::StateMachine`** — Step Functions.
- **Events** on a function: HttpApi, Api, S3, SQS, SNS, DynamoDB, Kinesis, Schedule, etc.

## Template example

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Resources:
  HelloFn:
    Type: AWS::Serverless::Function
    Properties:
      Runtime: nodejs20.x
      Handler: index.handler
      CodeUri: ./src/hello
      MemorySize: 256
      Timeout: 10
      Environment:
        Variables: { TABLE: !Ref OrdersTable }
      Policies:
        - DynamoDBCrudPolicy: { TableName: !Ref OrdersTable }
      Events:
        Api:
          Type: HttpApi
          Properties: { Path: /hello, Method: get }

  OrdersTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey: { Name: orderId, Type: String }
```

## Usage

### Install

```bash
brew tap aws/tap
brew install aws-sam-cli
```

### Commands

```bash
sam init          # scaffold a new project
sam build         # build artifacts
sam deploy --guided   # first deploy
sam deploy        # subsequent
sam local invoke HelloFn -e events/api.json
sam local start-api      # run API GW + Lambdas locally
sam logs -n HelloFn --tail
sam sync --watch  # hot deploy on file change
```

## Real-world example

> A small serverless backend with 8 endpoints, DynamoDB, S3 trigger.
> - One `template.yaml`, ~150 lines.
> - `sam deploy` updates all functions in 2 minutes.
> - `sam local start-api` for local dev with hot reload.

## Pricing

- **SAM is free.** You pay for the deployed resources.

## SAM vs CDK

- **SAM** — best for pure serverless apps. Smaller learning curve. YAML.
- **CDK** — best when you have a mix (VPC, ECS, RDS, IAM complexity) or want real code.

Many teams use SAM for Lambda-heavy services and CDK for everything else.

## Gotchas

- **`Transform` resources are SAM-only.** A SAM template only deploys correctly via the SAM CLI or CloudFormation (which auto-handles the transform).
- **Layers + dependencies** — managing Python/Node dependency layers is doable but fiddly.
- **`sam local`** uses Docker; may not match runtime exactly. Test in AWS for tricky cases.
- **`sam sync`** is fast but skips drift checks — re-run `sam deploy` for safety.

## Related

- [Lambda](../01-compute/lambda.md)
- [API Gateway](../04-networking/api-gateway.md)
- [CDK](./cdk.md)
- [CloudFormation](./cloudformation.md)
