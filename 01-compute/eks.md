# EKS — Elastic Kubernetes Service

**TL;DR** — Managed Kubernetes. AWS runs the control plane; you run workloads. $0.10/hr per cluster + node costs.

## What it is

A Kubernetes-as-a-Service. AWS hosts and operates the highly-available K8s control plane (etcd, API server, scheduler). You attach worker nodes (EC2, Fargate, or both) and `kubectl apply` as usual.

## Why it exists

K8s is complex to run yourself — etcd backups, upgrades, certificates, HA. EKS handles all that. Standard K8s API means everything in the K8s ecosystem (Helm charts, operators, Istio, ArgoCD) just works.

## Key concepts

- **Cluster** — the control plane. ~$72/mo per cluster.
- **Node group** — EC2 worker nodes managed by EKS.
- **Fargate profile** — serverless pods (no EC2 to manage).
- **Add-ons** — managed installs of CoreDNS, kube-proxy, VPC CNI, EBS CSI driver.
- **VPC CNI** — assigns VPC IPs to pods directly.
- **IRSA (IAM Roles for Service Accounts)** — pod-level AWS permissions.
- **Pod Identity** — newer, simpler alternative to IRSA.
- **EKS Auto Mode** — fully managed nodes + Karpenter, less config (2024+).
- **Karpenter** — fast, flexible node autoscaler (replaces Cluster Autoscaler for many).

## When to pick EKS over ECS

Choose EKS when:
- You already use K8s (skills, manifests, Helm charts).
- You want multi-cloud portability.
- You need K8s ecosystem (Istio, Argo, Knative, Crossplane).
- Complex scheduling / advanced workloads.

Choose ECS when:
- You just want to run containers behind a load balancer.
- Small team, no K8s skills.
- Tight AWS integration is fine.

## Real-world example

> A fintech team migrates from on-prem K8s to AWS. They lift their Helm charts to EKS with Karpenter and Fargate profiles for steady workloads. ArgoCD does GitOps deploys from their GitHub repo.

## Usage

### Create a cluster with eksctl (easiest)

```bash
brew install eksctl awscli kubectl

eksctl create cluster \
  --name prod \
  --region ap-south-1 \
  --version 1.30 \
  --nodegroup-name workers \
  --node-type m6i.large \
  --nodes 3 --nodes-min 2 --nodes-max 10 \
  --with-oidc \
  --managed
```

This takes ~15 minutes. Configures kubeconfig automatically.

### Verify

```bash
kubectl get nodes
kubectl get pods -A
```

### Deploy a workload

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: api }
spec:
  replicas: 3
  selector: { matchLabels: { app: api } }
  template:
    metadata: { labels: { app: api } }
    spec:
      serviceAccountName: api-sa  # uses IRSA for AWS access
      containers:
      - name: api
        image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/api:1.0
        ports: [{ containerPort: 8080 }]
---
apiVersion: v1
kind: Service
metadata: { name: api }
spec:
  type: LoadBalancer  # creates an NLB
  selector: { app: api }
  ports: [{ port: 80, targetPort: 8080 }]
```

```bash
kubectl apply -f api.yaml
kubectl get svc api  # shows the ELB DNS
```

### IRSA — pod-level IAM

```bash
eksctl create iamserviceaccount \
  --cluster prod --namespace default --name api-sa \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess \
  --approve
```

Now pods that use `serviceAccountName: api-sa` can call DynamoDB without static keys.

### CDK

```ts
const cluster = new eks.Cluster(this, 'Cluster', {
  version: eks.KubernetesVersion.V1_30,
  defaultCapacity: 0,  // skip default, use Fargate
});
cluster.addFargateProfile('Default', {
  selectors: [{ namespace: 'default' }],
});
```

## Add-ons (always install)

- **VPC CNI** — pod networking.
- **kube-proxy** — service routing.
- **CoreDNS** — cluster DNS.
- **EBS CSI** — persistent volumes on EBS.
- **AWS Load Balancer Controller** — provisions ALB/NLB for Ingress/Service.

Install via:
```bash
aws eks create-addon --cluster-name prod --addon-name vpc-cni
```

## Pricing

- **Control plane:** $0.10/hr per cluster ≈ **$72/mo**.
- **Workers:** standard EC2/Fargate pricing.
- **NAT gateway** — usually needed for pulling images, ~$32/mo per AZ.
- **Load balancers** — ~$16-25/mo per ALB/NLB.

Bare-minimum prod cluster ≈ **$150-300/mo** before workloads.

## Upgrades

EKS supports 4 K8s minor versions at a time. New version every ~3 months. Upgrade in order:
1. Control plane (one minor version up).
2. Add-ons.
3. Node groups (one minor up; recreate or rolling-update).
4. Workloads (check deprecated APIs).

## Gotchas

- **Pod IP exhaustion** — VPC CNI gives each pod a real VPC IP. Small subnets fill up fast. Use prefix delegation or secondary CIDR.
- **`kubectl` access requires IAM mapping** — see `aws-auth` ConfigMap or new EKS Access Entries (preferred now).
- **Cluster upgrade is one-way** — you can't downgrade.
- **Load balancers cost $$.** Use one Ingress + path-based routing instead of one LB per service.
- **Fargate has limits** — no DaemonSets, no privileged containers, no host network.
- **You pay $72/mo per cluster** before any workload. Consolidate dev/staging when possible.

## Related

- [ECS](./ecs.md) — simpler container service
- [Fargate](./fargate.md) — serverless compute for pods
- [ECR](#) — image registry
- [App Mesh](#) — service mesh (legacy now)
