# Kendra

**TL;DR** — Enterprise search with ML re-ranking. Indexes SharePoint, Confluence, ServiceNow, Salesforce, S3, RDS, plus 30+ connectors. Returns ranked passages, not just hits. Pricey.

## What it is

A managed semantic search engine for enterprise content. Unlike OpenSearch (you build the index + ranking), Kendra ingests via prebuilt connectors and does ML-based relevance + passage extraction + FAQ matching out of the box.

## Key concepts

- **Index** — top-level. Two editions: Developer (cheap, dev only) and Enterprise (prod).
- **Data source** — connector to S3 / SharePoint / Confluence / Salesforce / etc.
- **Document attributes / metadata** — used for filtering + boosting.
- **FAQ** — curated Q/A pairs that get priority matches.
- **Query API** — `/query` returns ranked results + extracted answers.
- **Access control** — query-time user/group filters honor source-system ACLs.

## Real-world example

> Internal IT help portal:
> - Connectors: SharePoint (docs), Confluence (wiki), ServiceNow (KB articles).
> - Curated FAQs for top 50 questions.
> - When employee searches "how do I reset MFA", Kendra returns the exact paragraph from the right wiki page.

## Usage

```js
import { KendraClient, QueryCommand } from "@aws-sdk/client-kendra";
const kendra = new KendraClient({ region: "us-east-1" });

const res = await kendra.send(new QueryCommand({
  IndexId: "<index-id>",
  QueryText: "refund policy",
  UserContext: { UserId: "u_42", Groups: ["staff"] },
}));
for (const r of res.ResultItems) {
  console.log(r.Type, r.DocumentTitle?.Text, r.DocumentExcerpt?.Text);
}
```

## Pricing

- **Developer Edition:** $1.125/hr (~$800/mo). Up to 10k docs, 4k queries/day.
- **Enterprise Edition:** $1.40/hr (~$1,000/mo) per index, plus connector hours.

Yes, Kendra is **expensive**. For small/medium teams, OpenSearch + an LLM is usually cheaper.

## Kendra vs OpenSearch vs Bedrock RAG

| | Kendra | OpenSearch | Bedrock KB + LLM |
|---|---|---|---|
| Setup effort | Lowest (connectors) | Higher | Mid |
| Quality | Good ML re-ranking | Depends on your work | Best for conversational |
| Cost | High | Medium | Pay-per-token |
| Use | Enterprise search w/ ACLs | Custom search | Chatbot / Q&A over docs |

## Gotchas

- **Cost is significant** — only justified at scale or when connector convenience matters.
- **Ingestion lag** — connectors sync on schedule, not instant.
- **ACL accuracy** depends on the source — verify per-user filtering.
- **Region availability is limited** — verify before adoption.

## Related

- [OpenSearch](../10-analytics/opensearch.md)
- [Bedrock Knowledge Bases](./bedrock.md)
- [Q Business (formerly Q for Business)](#) — Bedrock-powered, often replacing Kendra for new builds
