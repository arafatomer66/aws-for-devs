# SageMaker

**TL;DR** — Full-stack ML platform. Train models, host endpoints, build pipelines, label data. Sprawling — has 30+ sub-features. Most teams use a subset.

## What it is

SageMaker = "Everything ML on AWS." Key pieces:
- **Studio** — JupyterLab-like IDE in the browser.
- **Training jobs** — managed training on EC2/GPU/Trainium.
- **Endpoints** — host trained models for inference (real-time or batch).
- **Pipelines** — MLOps DAGs.
- **Feature Store** — managed feature catalog.
- **Ground Truth** — data labeling (with human workforce options).
- **Canvas** — no-code AutoML.
- **JumpStart** — pretrained model marketplace.
- **HyperPod** — managed cluster for distributed training.
- **Model Cards / Model Registry** — governance.
- **Clarify** — bias and explainability checks.
- **Edge Manager** — deploy to edge devices.

## Why it exists

End-to-end ML platform without you running infrastructure. Replaces a stack of: K8s + Argo + MLflow + Jupyter + Prometheus + Triton.

## Real-world example

> Fraud detection model:
> - Data scientists work in SageMaker Studio notebooks against Athena + S3 data.
> - Training job on `ml.p4d.24xlarge` (8× A100) for hyperparameter tuning.
> - Output model registered in Model Registry, approved by tech lead.
> - SageMaker Pipeline auto-deploys to a real-time endpoint behind API Gateway.
> - Model Monitor checks for data drift; alerts if features shift.

## Usage

### Train a model (Python SDK)

```python
from sagemaker.estimator import Estimator
est = Estimator(
    image_uri="763104351884.dkr.ecr.ap-south-1.amazonaws.com/pytorch-training:2.3.0-gpu-py311",
    role="arn:aws:iam::..:role/SageMakerExec",
    instance_count=1,
    instance_type="ml.g5.xlarge",
    output_path="s3://sd-ml/output/",
    hyperparameters={ "epochs": 5, "lr": 0.001 },
)
est.fit({"train":"s3://sd-ml/train/", "val":"s3://sd-ml/val/"})
```

### Deploy as a real-time endpoint

```python
predictor = est.deploy(initial_instance_count=1, instance_type="ml.g5.xlarge")
result = predictor.predict({"features": [...]})
```

### Serverless / Async endpoints

- **Serverless** — auto-scales to zero, $/inference.
- **Async** — for big payloads, queues + returns S3 result.
- **Batch Transform** — process large datasets offline.
- **Real-time** — always-on instance.

## SageMaker vs Bedrock

- **Bedrock** — managed FMs; you don't train them. Best for using LLMs / FMs.
- **SageMaker** — you bring/train/host your own models. Best for custom ML.

Many teams use both: Bedrock for LLMs, SageMaker for ranking/recommendation/CV/forecasting.

## Pricing

- Training: per-second of instance time.
  - `ml.g5.xlarge` ~ $1.50/hr.
  - `ml.p4d.24xlarge` ~ $32/hr.
- Inference endpoints: per-hour of provisioned instance (real-time) or per-inference (serverless).
- Notebook instances / Studio Apps: per-hour.
- Storage on EFS attached to Studio: ~$0.30/GB-mo.

## Gotchas

- **Endpoint instances run 24/7** — auto-stop them if idle, or use Serverless.
- **Spot training** is huge cost savings — supported, just enable.
- **Built-in algorithms** (XGBoost, Linear Learner, etc.) are great starting points.
- **GPU instance availability** can be tight in popular regions — try multiple.
- **Studio costs add up** if many notebooks left running.

## Related

- [Bedrock](./bedrock.md)
- [Q Developer](./q-developer.md)
- [Glue](../10-analytics/glue.md) — feature engineering pipelines
