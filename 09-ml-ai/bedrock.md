# Amazon Bedrock

**TL;DR** — Managed foundation models (FMs) via one API. Claude (Anthropic), Llama (Meta), Titan (Amazon), Mistral, Cohere, AI21, Stability. Pay per token, no GPUs to manage.

## What it is

A serverless gateway to multiple LLM/embedding/image-generation providers. Same SDK, switch models with a parameter. Plus features: Knowledge Bases (RAG), Agents (tool use), Guardrails, Model Evaluation, Custom Models (fine-tune).

## Available model families (2026)

- **Anthropic Claude** — `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5`. State-of-the-art reasoning/coding.
- **Amazon Titan / Nova** — text, embeddings, image.
- **Meta Llama** — open-weights.
- **Mistral / Mixtral**.
- **Cohere** — Command, Embed, Rerank.
- **AI21** — Jamba.
- **Stability AI** — Stable Diffusion XL.

Availability varies by region — check before committing to a region.

## Key concepts

- **Foundation Model (FM)** — pretrained, callable directly.
- **Provisioned Throughput** — reserve capacity for guaranteed throughput.
- **Custom model** — fine-tuned on your data; requires Provisioned Throughput.
- **Knowledge Bases** — RAG-as-a-service: point at S3, get a vector store + retrieval API.
- **Agents** — multi-step tool-using LLM workflows.
- **Guardrails** — filter unsafe inputs/outputs (PII, profanity, topics).
- **Model Evaluation** — A/B test models on your prompts.

## Real-world example

> ShareDeal customer-support chatbot:
> - Bedrock Knowledge Base over the FAQ S3 bucket.
> - Claude Sonnet 4.6 for responses; cheaper Haiku for classification.
> - Guardrail: blocks profanity, redacts PII before model sees it.
> - Latency: ~1.5 s for typical answer.
> - Cost: ~$0.003 per conversation.

## Usage

### Invoke model (Node, Claude)

```js
import { BedrockRuntimeClient, InvokeModelCommand } from "@aws-sdk/client-bedrock-runtime";
const br = new BedrockRuntimeClient({ region: "ap-south-1" });

const res = await br.send(new InvokeModelCommand({
  modelId: "anthropic.claude-sonnet-4-6-20260101-v1:0",
  contentType: "application/json",
  accept: "application/json",
  body: JSON.stringify({
    anthropic_version: "bedrock-2023-05-31",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Summarize this order in one line." }],
  }),
}));
const body = JSON.parse(new TextDecoder().decode(res.body));
console.log(body.content[0].text);
```

### Streaming

Use `InvokeModelWithResponseStreamCommand` to stream tokens.

### Knowledge Bases (RAG)

```bash
# 1. Create vector store (OpenSearch Serverless / Aurora pgvector / Pinecone / Redis)
# 2. Create KB pointing at it + an S3 data source
aws bedrock-agent create-knowledge-base ...
aws bedrock-agent create-data-source --knowledge-base-id <id> --name docs --data-source-configuration '{...S3 source...}'
aws bedrock-agent start-ingestion-job --knowledge-base-id <id> --data-source-id <id>

# 3. Query
aws bedrock-agent-runtime retrieve-and-generate \
  --input '{"text":"What is our refund policy?"}' \
  --retrieve-and-generate-configuration '{
    "type":"KNOWLEDGE_BASE",
    "knowledgeBaseConfiguration":{
      "knowledgeBaseId":"<id>",
      "modelArn":"arn:aws:bedrock:..:foundation-model/anthropic.claude-sonnet-4-6-..."
    }
  }'
```

### Guardrail

Create a guardrail, then pass `guardrailIdentifier` + `guardrailVersion` in your InvokeModel call.

### Prompt caching (Claude)

For repeated long context (system prompt, docs), cache by adding cache control markers in messages. Reduces cost dramatically.

## Pricing

Per-token, varies by model. Approx (Claude Sonnet 4.6):
- Input: $3/M tokens.
- Output: $15/M tokens.
- Prompt caching: ~90% off cached input tokens after first hit.

Haiku is ~10x cheaper than Sonnet, much faster.

## Bedrock vs SageMaker vs OpenAI direct

| | Bedrock | SageMaker | OpenAI API |
|---|---|---|---|
| Hosting | AWS-managed FM | You bring/train models | Their cloud |
| Models | Claude/Llama/Titan/etc. | Any | GPT-4o, o1 |
| AWS integration | Native | Native | None |
| Best for | Quick LLM use in AWS | Custom training/hosting | When you specifically want OpenAI |

## Gotchas

- **Model availability is per-region.** Some models only in `us-east-1` / `us-west-2`.
- **You must explicitly enable model access** in the console — defaults are off.
- **Quota limits per region/account** — request increase for production.
- **Provisioned Throughput is pricey** — usually only justified for high-volume custom-fine-tuned models.
- **Knowledge Base vector store** — newer "Vector store" options vary by region.
- **Streaming**: parse SSE-like chunks.
- **Cost surprises** — caps via Bedrock Cost Explorer + Budgets.

## Related

- [SageMaker](./sagemaker.md)
- [OpenSearch (vector store)](../10-analytics/opensearch.md)
- [Aurora pgvector](../03-database/aurora.md)
