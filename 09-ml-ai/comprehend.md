# Comprehend

**TL;DR** — NLP API. Sentiment, entities, key phrases, language detection, syntax, topic modeling, PII detection, custom classifiers. For tabular NLP, not LLM-grade tasks.

## What it does

- **Sentiment** — positive / negative / neutral / mixed.
- **Entities** — `PERSON`, `LOCATION`, `ORGANIZATION`, `DATE`, etc.
- **Key phrases** — important phrases per doc.
- **Language detection.**
- **PII detection / redaction** — emails, SSNs, addresses.
- **Custom classification** — train your own classifier with labels.
- **Custom entity recognition** — domain-specific NER.
- **Topic modeling** — LDA on a doc set.

## Real-world example

> Customer feedback pipeline:
> - Daily batch: pull last 24h of reviews from RDS.
> - Comprehend `BatchDetectSentiment` + `BatchDetectKeyPhrases`.
> - Store in DynamoDB; dashboard in QuickSight: "% negative reviews", "trending complaints."

## Usage

```js
import { ComprehendClient, DetectSentimentCommand, DetectEntitiesCommand } from "@aws-sdk/client-comprehend";
const cp = new ComprehendClient({ region: "ap-south-1" });

const { Sentiment } = await cp.send(new DetectSentimentCommand({ Text: "The delivery was super fast!", LanguageCode: "en" }));
const { Entities } = await cp.send(new DetectEntitiesCommand({ Text: "Arif from Dhaka ordered on Friday.", LanguageCode: "en" }));
```

## Pricing

- ~$0.0001 per unit (100 chars) for standard APIs.
- Custom classifiers higher.

## Comprehend vs Bedrock LLMs

- **Comprehend** — cheap, predictable, structured outputs. Good for batch.
- **LLM (Bedrock Claude/Haiku)** — flexible, can handle "extract anything I describe in this prompt." More expensive, slower.

For high-volume known-structure NLP, Comprehend is much cheaper.

## Gotchas

- **Languages supported vary by API** — sentiment is broad; some are EN-only.
- **5,000 byte max** per single-doc call; batch APIs handle up to 25 docs at once.
- **Custom models** take time to train and require labeled data.

## Related

- [Bedrock](./bedrock.md)
- [Translate](./translate.md)
