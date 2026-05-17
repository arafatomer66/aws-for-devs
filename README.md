# AWS for Developers

> Everything a developer needs to know about AWS, in one place.

A practical, no-fluff reference for developers. Each service has its own file with: what it is, why it exists, key concepts, a real-world example, code/CLI usage, pricing model, gotchas, and related services.

If you're new to AWS, start with **[00-foundations](./00-foundations/)**. If you know AWS but need a refresher on a service, jump to its file from the index below.

---

## Table of Contents

### 00. Foundations
The mental model. Read these first.
- [Global Infrastructure](./00-foundations/01-global-infrastructure.md) — Regions, AZs, Edge Locations, Local Zones
- [Shared Responsibility Model](./00-foundations/02-shared-responsibility.md) — What AWS secures vs. what you secure
- [Well-Architected Framework](./00-foundations/03-well-architected.md) — 6 pillars every workload should follow
- [Accounts, Billing, Free Tier](./00-foundations/04-accounts-billing.md) — Account setup, MFA, free tier limits
- [Pricing Models](./00-foundations/05-pricing-models.md) — On-demand, Reserved, Spot, Savings Plans
- [Regions & Service Availability](./00-foundations/06-regions.md) — Which services run where

### 01. Compute
Where your code runs.
- [EC2](./01-compute/ec2.md) — Virtual machines, the foundation of AWS compute
- [Lambda](./01-compute/lambda.md) — Serverless functions, pay-per-invocation
- [ECS](./01-compute/ecs.md) — Managed Docker container orchestration
- [EKS](./01-compute/eks.md) — Managed Kubernetes
- [Fargate](./01-compute/fargate.md) — Serverless containers (no EC2 to manage)
- [AWS Batch](./01-compute/batch.md) — Batch computing jobs at any scale
- [Lightsail](./01-compute/lightsail.md) — Simple VPS, flat pricing
- [Elastic Beanstalk](./01-compute/elastic-beanstalk.md) — PaaS for web apps
- [App Runner](./01-compute/app-runner.md) — Fully managed container web apps

### 02. Storage
Where your data lives at rest.
- [S3](./02-storage/s3.md) — Object storage, the most-used AWS service
- [EBS](./02-storage/ebs.md) — Block storage for EC2 (think: a virtual disk)
- [EFS](./02-storage/efs.md) — Managed NFS file system
- [FSx](./02-storage/fsx.md) — Managed Windows/Lustre/NetApp file systems
- [S3 Glacier](./02-storage/glacier.md) — Archival storage (cents per GB-year)
- [Storage Gateway](./02-storage/storage-gateway.md) — Hybrid cloud storage

### 03. Database
- [RDS](./03-database/rds.md) — Managed relational DB (Postgres, MySQL, MariaDB, Oracle, SQL Server)
- [Aurora](./03-database/aurora.md) — AWS's cloud-native MySQL/Postgres-compatible engine
- [DynamoDB](./03-database/dynamodb.md) — Managed NoSQL key-value/document, single-digit ms latency
- [ElastiCache](./03-database/elasticache.md) — Managed Redis & Memcached
- [Redshift](./03-database/redshift.md) — Data warehouse for analytics on TB-PB scale
- [DocumentDB](./03-database/documentdb.md) — MongoDB-compatible
- [Neptune](./03-database/neptune.md) — Graph database
- [Timestream](./03-database/timestream.md) — Time-series database
- [Keyspaces](./03-database/keyspaces.md) — Managed Cassandra
- [MemoryDB](./03-database/memorydb.md) — Redis-compatible, durable in-memory DB

### 04. Networking & Content Delivery
- [VPC](./04-networking/vpc.md) — Virtual Private Cloud, your isolated network
- [Route 53](./04-networking/route53.md) — DNS, domain registration, health checks
- [CloudFront](./04-networking/cloudfront.md) — CDN at the edge
- [API Gateway](./04-networking/api-gateway.md) — Managed REST/HTTP/WebSocket APIs
- [Elastic Load Balancing](./04-networking/elb.md) — ALB, NLB, GWLB
- [Direct Connect](./04-networking/direct-connect.md) — Dedicated network from on-prem to AWS
- [Transit Gateway](./04-networking/transit-gateway.md) — Hub for VPC/on-prem connectivity
- [VPN Gateway](./04-networking/vpn.md) — Site-to-site and client VPN
- [PrivateLink](./04-networking/privatelink.md) — Private access to services without internet
- [Global Accelerator](./04-networking/global-accelerator.md) — Anycast IPs over AWS backbone

### 05. Security, Identity & Compliance
- [IAM](./05-security-iam/iam.md) — Users, roles, policies — the core of AWS security
- [KMS](./05-security-iam/kms.md) — Managed encryption keys
- [Secrets Manager](./05-security-iam/secrets-manager.md) — Store/rotate secrets
- [Parameter Store](./05-security-iam/parameter-store.md) — SSM Parameter Store for config
- [Cognito](./05-security-iam/cognito.md) — User pools & identity pools for app auth
- [WAF](./05-security-iam/waf.md) — Web Application Firewall
- [Shield](./05-security-iam/shield.md) — DDoS protection
- [GuardDuty](./05-security-iam/guardduty.md) — Threat detection
- [Security Hub](./05-security-iam/security-hub.md) — Aggregate security findings
- [ACM](./05-security-iam/acm.md) — Free TLS certificates
- [IAM Identity Center](./05-security-iam/identity-center.md) — SSO across accounts (was AWS SSO)

### 06. Messaging & Integration
- [SQS](./06-messaging-integration/sqs.md) — Managed message queues
- [SNS](./06-messaging-integration/sns.md) — Pub/sub topics, fan-out
- [EventBridge](./06-messaging-integration/eventbridge.md) — Event bus, the modern event-driven backbone
- [Kinesis Data Streams](./06-messaging-integration/kinesis.md) — Real-time streaming
- [MSK](./06-messaging-integration/msk.md) — Managed Kafka
- [Step Functions](./06-messaging-integration/step-functions.md) — Visual workflows / state machines
- [MQ](./06-messaging-integration/mq.md) — Managed ActiveMQ / RabbitMQ
- [AppSync](./06-messaging-integration/appsync.md) — Managed GraphQL

### 07. DevOps & Infrastructure as Code
- [CloudFormation](./07-devops-iac/cloudformation.md) — Native IaC (YAML/JSON)
- [CDK](./07-devops-iac/cdk.md) — IaC in TypeScript/Python/Java/etc.
- [SAM](./07-devops-iac/sam.md) — Serverless Application Model
- [CodeCommit](./07-devops-iac/codecommit.md) — Managed Git
- [CodeBuild](./07-devops-iac/codebuild.md) — Managed CI build
- [CodeDeploy](./07-devops-iac/codedeploy.md) — Automated deployments
- [CodePipeline](./07-devops-iac/codepipeline.md) — CI/CD orchestration
- [CodeArtifact](./07-devops-iac/codeartifact.md) — Managed artifact repo (npm, Maven, PyPI, etc.)
- [Systems Manager](./07-devops-iac/systems-manager.md) — Patch/run/inventory fleet of servers

### 08. Monitoring & Observability
- [CloudWatch](./08-monitoring-observability/cloudwatch.md) — Metrics, logs, alarms, dashboards
- [X-Ray](./08-monitoring-observability/x-ray.md) — Distributed tracing
- [CloudTrail](./08-monitoring-observability/cloudtrail.md) — API audit log of who did what
- [Config](./08-monitoring-observability/config.md) — Track resource configuration over time
- [Managed Grafana / Prometheus](./08-monitoring-observability/grafana-prometheus.md)

### 09. ML & AI
- [Bedrock](./09-ml-ai/bedrock.md) — Managed foundation models (Claude, Llama, Titan, etc.)
- [SageMaker](./09-ml-ai/sagemaker.md) — Build/train/deploy ML models
- [Rekognition](./09-ml-ai/rekognition.md) — Image & video analysis
- [Comprehend](./09-ml-ai/comprehend.md) — NLP, sentiment, entities
- [Polly](./09-ml-ai/polly.md) — Text-to-speech
- [Transcribe](./09-ml-ai/transcribe.md) — Speech-to-text
- [Translate](./09-ml-ai/translate.md) — Neural machine translation
- [Textract](./09-ml-ai/textract.md) — OCR & form/table extraction
- [Q Developer](./09-ml-ai/q-developer.md) — AWS's coding assistant (was CodeWhisperer)

### 10. Analytics
- [Athena](./10-analytics/athena.md) — Serverless SQL on S3 data
- [Glue](./10-analytics/glue.md) — Serverless ETL & data catalog
- [EMR](./10-analytics/emr.md) — Managed Hadoop / Spark / Hive
- [QuickSight](./10-analytics/quicksight.md) — Managed BI dashboards
- [OpenSearch](./10-analytics/opensearch.md) — Managed Elasticsearch fork
- [Kinesis Data Firehose](./10-analytics/firehose.md) — Streaming load to S3/Redshift/OpenSearch
- [Lake Formation](./10-analytics/lake-formation.md) — Build & govern data lakes
- [MSK Connect](./10-analytics/msk-connect.md) — Kafka Connect

### 11. Cost & Account Management
- [Cost Explorer](./11-cost-management/cost-explorer.md) — Analyze AWS spend
- [Budgets](./11-cost-management/budgets.md) — Alert when forecasted to overspend
- [Organizations](./11-cost-management/organizations.md) — Multi-account management
- [Control Tower](./11-cost-management/control-tower.md) — Landing zone for multi-account
- [Trusted Advisor](./11-cost-management/trusted-advisor.md) — Best-practice checks
- [Compute Optimizer](./11-cost-management/compute-optimizer.md) — Right-sizing recommendations

### 12. Migration & Hybrid
- [DMS](./12-migration/dms.md) — Database Migration Service
- [DataSync](./12-migration/datasync.md) — Bulk data move on/off AWS
- [Snow Family](./12-migration/snow-family.md) — Snowcone/Snowball/Snowmobile, physical data transfer
- [MGN](./12-migration/mgn.md) — Application Migration Service (rehost)
- [Outposts](./12-migration/outposts.md) — AWS hardware in your data center

### 13. Developer Tools (CLI, SDKs, Local Dev)
- [AWS CLI](./13-developer-tools/cli.md) — Command-line interface
- [SDKs](./13-developer-tools/sdks.md) — JS/Python/Java/Go/Rust/.NET/PHP/Ruby
- [CDK](./13-developer-tools/cdk-quickstart.md) — Quickstart, same concept as 07
- [Amplify](./13-developer-tools/amplify.md) — Full-stack for mobile/web devs
- [CloudShell](./13-developer-tools/cloudshell.md) — Browser-based terminal
- [Location Service](./13-developer-tools/location-service.md) — Managed maps, geocoding, routing, geofences, trackers
- [LocalStack](./13-developer-tools/localstack.md) — Run AWS locally for dev/test (3rd party but ubiquitous)

---

## How to use this repo

1. **Read foundations** — without the mental model, the services look like a confusing zoo.
2. **Pick a use case** (e.g. "I want to deploy a web API"), then read the 2–4 service files that solve it.
3. **Copy the CLI/SDK snippets** in each file — they're written to be paste-and-run with minimal edits.
4. **Open the AWS docs** when you go deeper. This repo is a map; the docs are the territory.

## Common use-case recipes

Quick "which services do I combine for X" answers:

| I want to… | Reach for |
|---|---|
| Host a static website | S3 + CloudFront + Route 53 + ACM |
| Run a Node/Python API | Lambda + API Gateway, or ECS Fargate + ALB |
| Run a containerized app | ECS Fargate (simple) or EKS (k8s) |
| Store user uploads | S3 with presigned URLs |
| Add login to my app | Cognito User Pools (or roll your own with JWT) |
| Send transactional email | SES |
| Background jobs / async work | SQS + Lambda, or Step Functions for workflows |
| Real-time pub/sub | SNS (fan-out) or EventBridge (routing rules) |
| Stream events | Kinesis Data Streams or MSK (Kafka) |
| Run scheduled jobs (cron) | EventBridge Scheduler → Lambda/ECS |
| Run an LLM | Bedrock (managed) or SageMaker (custom) |
| Search across documents | OpenSearch |
| Show a map / geocode / route | Location Service (or Google Maps if UX-critical) |
| Data warehouse | Redshift, or Athena over S3 (cheaper) |
| Cache | ElastiCache (Redis) or DynamoDB DAX |
| CI/CD | CodePipeline + CodeBuild, or GitHub Actions → AWS |
| Secrets | Secrets Manager (with rotation) or Parameter Store (cheaper) |

## Conventions used in service files

Every service file follows the same structure:

- **TL;DR** — One sentence so you can skim
- **What it is** — Plain-English explanation
- **Why it exists** — The problem it solves
- **Key concepts** — Vocabulary you need to know
- **Real-world example** — A concrete scenario
- **Usage** — CLI / SDK / IaC snippets
- **Pricing model** — How you get billed
- **Gotchas** — Things that bite people
- **Related services** — What to read next

## Contributing

PRs welcome. Keep files concise — if it doesn't fit on one screen-and-a-bit, it's too long. Link to the AWS docs for deep dives.

## License

MIT — see [LICENSE](./LICENSE).
