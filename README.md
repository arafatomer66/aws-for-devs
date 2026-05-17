<div align="center">

<br />

<h1>AWS for Developers</h1>

<p>
  <strong>Everything a developer needs to know about AWS — in one place.</strong>
</p>

<p>
  <a href="#start-here"><img alt="Start here" src="https://img.shields.io/badge/START%20HERE-Foundations-2563eb?style=for-the-badge" /></a>
  <a href="#table-of-contents"><img alt="Browse services" src="https://img.shields.io/badge/BROWSE-117%20Services-f97316?style=for-the-badge" /></a>
  <a href="#common-use-case-recipes"><img alt="Recipes" src="https://img.shields.io/badge/RECIPES-30%2B-16a34a?style=for-the-badge" /></a>
</p>

<p>
  <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square" />
  <img alt="Categories" src="https://img.shields.io/badge/categories-16-blueviolet?style=flat-square" />
  <img alt="Pages" src="https://img.shields.io/badge/pages-117-orange?style=flat-square" />
  <img alt="PRs welcome" src="https://img.shields.io/badge/PRs-welcome-success?style=flat-square" />
</p>

<br />

<p><em>Skim it like a map. Reach for the AWS docs when you go deeper.</em></p>

</div>

---

## Why this repo exists

AWS docs are *exhaustive* — and exhausting. When you just need to ship something, you don't want a 60-page service guide; you want **the 3 paragraphs that matter**, **a working snippet**, and **the gotchas other developers wish they'd known**.

That's this repo. Every service page follows the same skim-friendly format. No filler.

> [!TIP]
> If this is your first time on AWS, jump to **[Start here](#start-here)**.
> If you know AWS and need a refresher on a service, find it in the **[Table of Contents](#table-of-contents)**.
> If you have a problem in mind ("how do I run a containerized app?"), check **[Recipes](#common-use-case-recipes)**.

---

## At a glance

<table>
  <tr>
    <td><strong>Services covered</strong></td>
    <td>~117</td>
  </tr>
  <tr>
    <td><strong>Foundation concepts</strong></td>
    <td>6</td>
  </tr>
  <tr>
    <td><strong>Categories</strong></td>
    <td>16</td>
  </tr>
  <tr>
    <td><strong>Page format</strong></td>
    <td>TL;DR → concepts → real example → CLI / SDK → pricing → gotchas → related</td>
  </tr>
  <tr>
    <td><strong>Page length</strong></td>
    <td>About one screen + a bit</td>
  </tr>
  <tr>
    <td><strong>License</strong></td>
    <td>MIT</td>
  </tr>
</table>

---

## Start here

New to AWS? These six pages give you the mental model. Read them in order; the rest of the repo assumes them.

| # | Page | What it covers |
|:-:|---|---|
| 1 | [Global Infrastructure](./00-foundations/01-global-infrastructure.md) | Regions, AZs, Edge Locations |
| 2 | [Shared Responsibility Model](./00-foundations/02-shared-responsibility.md) | What AWS secures vs. what you secure |
| 3 | [Well-Architected Framework](./00-foundations/03-well-architected.md) | The six pillars every workload should follow |
| 4 | [Accounts, Billing, Free Tier](./00-foundations/04-accounts-billing.md) | Day-1 setup, MFA, tagging strategy |
| 5 | [Pricing Models](./00-foundations/05-pricing-models.md) | On-Demand, Spot, Reserved, Savings Plans |
| 6 | [Regions & Service Availability](./00-foundations/06-regions.md) | Which services run where |

---

## Table of Contents

Each category collapsed for scanning. Click to expand.

<details open>
<summary><strong>Compute</strong> — 10 services — where your code runs</summary>

| Service | One-liner |
|---|---|
| [EC2](./01-compute/ec2.md) | Virtual machines, the foundation of AWS compute |
| [Lambda](./01-compute/lambda.md) | Serverless functions, pay-per-invocation |
| [ECS](./01-compute/ecs.md) | Managed Docker container orchestration |
| [EKS](./01-compute/eks.md) | Managed Kubernetes |
| [Fargate](./01-compute/fargate.md) | Serverless containers — no EC2 to manage |
| [ECR](./01-compute/ecr.md) | Private Docker registry |
| [AWS Batch](./01-compute/batch.md) | Batch computing at any scale |
| [Lightsail](./01-compute/lightsail.md) | Simple VPS with flat pricing |
| [Elastic Beanstalk](./01-compute/elastic-beanstalk.md) | PaaS for web apps |
| [App Runner](./01-compute/app-runner.md) | Fully managed container web apps |

</details>

<details>
<summary><strong>Storage</strong> — 7 services — where your data lives at rest</summary>

| Service | One-liner |
|---|---|
| [S3](./02-storage/s3.md) | Object storage — the most-used AWS service |
| [EBS](./02-storage/ebs.md) | Block storage for EC2 |
| [EFS](./02-storage/efs.md) | Managed NFS file system |
| [FSx](./02-storage/fsx.md) | Managed Windows / Lustre / NetApp file systems |
| [S3 Glacier](./02-storage/glacier.md) | Archival storage (cents per GB-year) |
| [Storage Gateway](./02-storage/storage-gateway.md) | Hybrid cloud storage |
| [AWS Backup](./02-storage/aws-backup.md) | Cross-service backup orchestration |

</details>

<details>
<summary><strong>Database</strong> — 10 services</summary>

| Service | One-liner |
|---|---|
| [RDS](./03-database/rds.md) | Managed Postgres / MySQL / MariaDB / Oracle / SQL Server |
| [Aurora](./03-database/aurora.md) | AWS's cloud-native MySQL / Postgres engine |
| [DynamoDB](./03-database/dynamodb.md) | Managed NoSQL, single-digit-ms latency |
| [ElastiCache](./03-database/elasticache.md) | Managed Redis & Memcached |
| [Redshift](./03-database/redshift.md) | Data warehouse for analytics |
| [DocumentDB](./03-database/documentdb.md) | MongoDB-compatible |
| [Neptune](./03-database/neptune.md) | Graph database |
| [Timestream](./03-database/timestream.md) | Time-series database |
| [Keyspaces](./03-database/keyspaces.md) | Managed Cassandra |
| [MemoryDB](./03-database/memorydb.md) | Durable Redis-compatible DB |

</details>

<details>
<summary><strong>Networking &amp; Content Delivery</strong> — 11 services</summary>

| Service | One-liner |
|---|---|
| [VPC](./04-networking/vpc.md) | Your isolated network on AWS |
| [Route 53](./04-networking/route53.md) | DNS, domain registration, health checks |
| [CloudFront](./04-networking/cloudfront.md) | CDN at the edge |
| [API Gateway](./04-networking/api-gateway.md) | Managed REST / HTTP / WebSocket APIs |
| [Elastic Load Balancing](./04-networking/elb.md) | ALB, NLB, GLB |
| [Direct Connect](./04-networking/direct-connect.md) | Dedicated network from on-prem to AWS |
| [Transit Gateway](./04-networking/transit-gateway.md) | Hub for VPC / on-prem connectivity |
| [VPN Gateway](./04-networking/vpn.md) | Site-to-site and Client VPN |
| [PrivateLink](./04-networking/privatelink.md) | Private access without internet |
| [Global Accelerator](./04-networking/global-accelerator.md) | Anycast IPs over AWS backbone |
| [Cloud Map](./04-networking/cloud-map.md) | Service discovery |

</details>

<details>
<summary><strong>Security, Identity &amp; Compliance</strong> — 13 services</summary>

| Service | One-liner |
|---|---|
| [IAM](./05-security-iam/iam.md) | Users, roles, policies — the core of AWS security |
| [KMS](./05-security-iam/kms.md) | Managed encryption keys |
| [Secrets Manager](./05-security-iam/secrets-manager.md) | Store & rotate secrets |
| [Parameter Store](./05-security-iam/parameter-store.md) | SSM Parameter Store for config |
| [Cognito](./05-security-iam/cognito.md) | User pools & identity for app auth |
| [WAF](./05-security-iam/waf.md) | Web Application Firewall |
| [Shield](./05-security-iam/shield.md) | DDoS protection |
| [GuardDuty](./05-security-iam/guardduty.md) | Threat detection |
| [Inspector](./05-security-iam/inspector.md) | Vulnerability scanning for EC2 / ECR / Lambda |
| [Macie](./05-security-iam/macie.md) | S3 sensitive-data discovery |
| [Security Hub](./05-security-iam/security-hub.md) | Aggregate security findings |
| [ACM](./05-security-iam/acm.md) | Free TLS certificates |
| [IAM Identity Center](./05-security-iam/identity-center.md) | SSO across accounts |

</details>

<details>
<summary><strong>Messaging &amp; Integration</strong> — 10 services</summary>

| Service | One-liner |
|---|---|
| [SQS](./06-messaging-integration/sqs.md) | Managed message queues |
| [SNS](./06-messaging-integration/sns.md) | Pub/sub topics, fan-out |
| [EventBridge](./06-messaging-integration/eventbridge.md) | Event bus — the modern event-driven backbone |
| [Kinesis Data Streams](./06-messaging-integration/kinesis.md) | Real-time streaming |
| [MSK](./06-messaging-integration/msk.md) | Managed Kafka |
| [Step Functions](./06-messaging-integration/step-functions.md) | Visual workflows / state machines |
| [MQ](./06-messaging-integration/mq.md) | Managed ActiveMQ / RabbitMQ |
| [AppSync](./06-messaging-integration/appsync.md) | Managed GraphQL |
| [SES](./06-messaging-integration/ses.md) | Transactional email |
| [Pinpoint](./06-messaging-integration/pinpoint.md) | Multi-channel customer engagement |

</details>

<details>
<summary><strong>DevOps &amp; Infrastructure as Code</strong> — 10 services</summary>

| Service | One-liner |
|---|---|
| [CloudFormation](./07-devops-iac/cloudformation.md) | Native IaC (YAML / JSON) |
| [CDK](./07-devops-iac/cdk.md) | IaC in real programming languages |
| [SAM](./07-devops-iac/sam.md) | Serverless Application Model |
| [CodeCommit](./07-devops-iac/codecommit.md) | Managed Git (deprecated for new) |
| [CodeBuild](./07-devops-iac/codebuild.md) | Managed CI build |
| [CodeDeploy](./07-devops-iac/codedeploy.md) | Automated deployments |
| [CodePipeline](./07-devops-iac/codepipeline.md) | CI/CD orchestration |
| [CodeArtifact](./07-devops-iac/codeartifact.md) | Managed artifact repo |
| [Systems Manager](./07-devops-iac/systems-manager.md) | Patch / Run Command / Session Manager |
| [AppConfig](./07-devops-iac/appconfig.md) | Feature flags & dynamic config |

</details>

<details>
<summary><strong>Monitoring &amp; Observability</strong> — 5 services</summary>

| Service | One-liner |
|---|---|
| [CloudWatch](./08-monitoring-observability/cloudwatch.md) | Metrics, logs, alarms, dashboards |
| [X-Ray](./08-monitoring-observability/x-ray.md) | Distributed tracing |
| [CloudTrail](./08-monitoring-observability/cloudtrail.md) | API audit log |
| [Config](./08-monitoring-observability/config.md) | Track resource configuration over time |
| [Managed Grafana / Prometheus](./08-monitoring-observability/grafana-prometheus.md) | OSS-compatible observability |

</details>

<details>
<summary><strong>ML &amp; AI</strong> — 11 services</summary>

| Service | One-liner |
|---|---|
| [Bedrock](./09-ml-ai/bedrock.md) | Managed foundation models (Claude, Llama, Titan, etc.) |
| [SageMaker](./09-ml-ai/sagemaker.md) | Build / train / deploy ML models |
| [Rekognition](./09-ml-ai/rekognition.md) | Image & video analysis |
| [Comprehend](./09-ml-ai/comprehend.md) | NLP — sentiment, entities, key phrases |
| [Polly](./09-ml-ai/polly.md) | Text-to-speech |
| [Transcribe](./09-ml-ai/transcribe.md) | Speech-to-text |
| [Translate](./09-ml-ai/translate.md) | Neural machine translation |
| [Textract](./09-ml-ai/textract.md) | OCR & form / table extraction |
| [Lex](./09-ml-ai/lex.md) | Conversational bots (Alexa-style) |
| [Kendra](./09-ml-ai/kendra.md) | Enterprise search with ML re-ranking |
| [Q Developer](./09-ml-ai/q-developer.md) | AWS's AI coding assistant |

</details>

<details>
<summary><strong>Analytics</strong> — 9 services</summary>

| Service | One-liner |
|---|---|
| [Athena](./10-analytics/athena.md) | Serverless SQL on S3 |
| [Glue](./10-analytics/glue.md) | Serverless ETL & data catalog |
| [EMR](./10-analytics/emr.md) | Managed Hadoop / Spark / Hive |
| [QuickSight](./10-analytics/quicksight.md) | Managed BI dashboards |
| [OpenSearch](./10-analytics/opensearch.md) | Managed Elasticsearch fork |
| [Kinesis Data Firehose](./10-analytics/firehose.md) | Streaming load to S3 / Redshift / OpenSearch |
| [Lake Formation](./10-analytics/lake-formation.md) | Build & govern data lakes |
| [MSK Connect](./10-analytics/msk-connect.md) | Managed Kafka Connect |
| [MWAA](./10-analytics/mwaa.md) | Managed Apache Airflow |

</details>

<details>
<summary><strong>Cost &amp; Account Management</strong> — 6 services</summary>

| Service | One-liner |
|---|---|
| [Cost Explorer](./11-cost-management/cost-explorer.md) | Analyze AWS spend |
| [Budgets](./11-cost-management/budgets.md) | Alert on overspend |
| [Organizations](./11-cost-management/organizations.md) | Multi-account management |
| [Control Tower](./11-cost-management/control-tower.md) | Landing zone for multi-account |
| [Trusted Advisor](./11-cost-management/trusted-advisor.md) | Best-practice checks |
| [Compute Optimizer](./11-cost-management/compute-optimizer.md) | Right-sizing recommendations |

</details>

<details>
<summary><strong>Migration &amp; Hybrid</strong> — 5 services</summary>

| Service | One-liner |
|---|---|
| [DMS](./12-migration/dms.md) | Database Migration Service |
| [DataSync](./12-migration/datasync.md) | Bulk data move on / off AWS |
| [Snow Family](./12-migration/snow-family.md) | Physical data transfer |
| [MGN](./12-migration/mgn.md) | Application Migration Service |
| [Outposts](./12-migration/outposts.md) | AWS hardware in your data center |

</details>

<details>
<summary><strong>Developer Tools</strong> — 7 services</summary>

| Service | One-liner |
|---|---|
| [AWS CLI](./13-developer-tools/cli.md) | Command-line interface |
| [SDKs](./13-developer-tools/sdks.md) | JS / Python / Java / Go / Rust / .NET |
| [CDK Quickstart](./13-developer-tools/cdk-quickstart.md) | Quick-start to the deeper CDK guide |
| [Amplify](./13-developer-tools/amplify.md) | Full-stack for mobile / web |
| [CloudShell](./13-developer-tools/cloudshell.md) | Browser-based terminal |
| [Location Service](./13-developer-tools/location-service.md) | Maps, geocoding, routing, geofences |
| [LocalStack](./13-developer-tools/localstack.md) | Run AWS locally for dev / test |

</details>

<details>
<summary><strong>IoT</strong> — 1 service</summary>

| Service | One-liner |
|---|---|
| [IoT Core](./14-iot/iot-core.md) | Managed MQTT broker + device shadow + rules |

</details>

<details>
<summary><strong>Media</strong> — 3 services</summary>

| Service | One-liner |
|---|---|
| [MediaConvert](./15-media/mediaconvert.md) | Batch video transcoding |
| [MediaLive](./15-media/medialive.md) | Live video encoding |
| [Chime SDK](./15-media/chime-sdk.md) | Embed audio / video calls into your app |

</details>

---

## Common use-case recipes

Quick "**which services do I combine for X**" answers.

| I want to… | Reach for |
|---|---|
| Host a static website | S3 + CloudFront + Route 53 + ACM |
| Run a Node / Python API | Lambda + API Gateway, or ECS Fargate + ALB |
| Run a containerized app | ECS Fargate (simple) or EKS (Kubernetes) |
| Store user uploads | S3 with presigned URLs |
| Add login to my app | Cognito User Pools (or roll your own JWT) |
| Send transactional email | SES |
| Run a marketing campaign | Pinpoint |
| Background jobs / async work | SQS + Lambda, or Step Functions for workflows |
| Real-time pub/sub | SNS (fan-out) or EventBridge (routing rules) |
| Stream events | Kinesis Data Streams or MSK (Kafka) |
| Run scheduled jobs (cron) | EventBridge Scheduler → Lambda / ECS |
| Run an LLM | Bedrock (managed) or SageMaker (custom) |
| Build a chatbot | Bedrock + Knowledge Bases, or Lex for structured flows |
| Search across documents | OpenSearch (custom) or Kendra (enterprise) |
| Data warehouse | Redshift, or Athena over S3 (cheaper) |
| Cache | ElastiCache (Redis) or DynamoDB DAX |
| CI/CD | GitHub Actions → AWS via OIDC, or CodePipeline + CodeBuild |
| Secrets | Secrets Manager (rotation) or Parameter Store (cheaper) |
| Feature flags | AppConfig |
| Show a map / geocode / route | Location Service (or Google Maps if UX-critical) |
| Backups across services | AWS Backup |
| Connect IoT devices | IoT Core (MQTT) |
| Transcode video | MediaConvert (batch) or MediaLive (live) |
| Embed video calls | Chime SDK |
| Find PII in S3 | Macie |
| Scan for vulnerabilities | Inspector |
| Audit log of API calls | CloudTrail |
| Distributed tracing | X-Ray (or OpenTelemetry → ADOT) |

---

## How to use this repo

> [!NOTE]
> This repo is a **map**, not the territory. Use it to orient yourself; the AWS docs are the source of truth.

1. **Read the foundations.** Without the mental model, services look unrelated.
2. **Pick a use case** ("I want to deploy a web API") — read the 2–4 service pages that solve it.
3. **Copy the snippets.** They're written to be paste-and-run with minimal edits.
4. **Skim the *Gotchas* section.** That's where the time-saving lives.
5. **Open the AWS docs** when you need to go deeper.

---

## Page template

Every service page follows the same skim-friendly structure:

```text
TL;DR              one sentence
What it is         plain-English explanation
Why it exists      the problem it solves
Key concepts       vocabulary you need to know
Real-world example a concrete scenario
Usage              CLI / SDK / IaC snippets
Pricing            how you get billed
Gotchas            things that bite people
Related            what to read next
```

---

## Contributing

Pull requests welcome. House style:

- **One file per service.** Use the page template above.
- **Concise.** If it doesn't fit on one screen + a bit, it's too long. Link to AWS docs for deep dives.
- **Concrete examples.** Vague hypotheticals don't stick — reference a real-shaped scenario.
- **Pasteable snippets.** Prefer code over prose where a snippet would do the job.
- **Update the README index** when you add a service.

---

<div align="center">

**Made for developers who'd rather ship than read 60 pages of docs.**

[MIT License](./LICENSE) — use freely.

</div>
