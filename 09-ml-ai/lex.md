# Amazon Lex

**TL;DR** — Conversational chatbot / IVR engine. Same tech as Alexa. Define intents + slots, hook to Lambda fulfillment. Voice and text channels. Often supplanted by Bedrock LLMs for new builds.

## What it is

A bot-builder for goal-oriented conversations: "book a flight", "check my order", "transfer money". You define **intents** (what the user wants), **slots** (params to collect), and a fulfillment Lambda. Lex handles NLU, slot filling, prompts, and dialog state.

## Key concepts

- **Bot** — a named conversational agent.
- **Intent** — user goal (`BookFlight`, `CheckOrderStatus`).
- **Utterance** — example phrase ("I want to fly to Dhaka tomorrow").
- **Slot** — variable extracted from utterance (`destination=Dhaka`, `date=tomorrow`).
- **Slot type** — built-in (city, date, number) or custom.
- **Fulfillment** — Lambda that does the actual work.
- **Channels** — Connect (call center), Twilio, Slack, Facebook, Genesys, your app.
- **Lex V2** — current; V1 is legacy.

## Lex vs Bedrock LLM agents

- **Lex** — structured, scoped, predictable. Great for IVR, customer support routing.
- **Bedrock Agents / Claude** — flexible, open-domain, tool-using. Costlier and less predictable.

Many teams now use LLMs for open Q&A and keep Lex for hard-edged flows (auth, transactions).

## Real-world example

> Call center IVR for ShareDeal:
> - Intent `CheckOrderStatus` with slot `OrderId`.
> - Fulfillment Lambda queries DynamoDB.
> - Channel: Amazon Connect (phone).
> - Falls back to a human agent if Lex can't resolve.

## Usage

Bot construction is mostly console-driven. The runtime API:

```js
import { LexRuntimeV2Client, RecognizeTextCommand } from "@aws-sdk/client-lex-runtime-v2";
const lex = new LexRuntimeV2Client({ region: "ap-south-1" });

const res = await lex.send(new RecognizeTextCommand({
  botId: "...", botAliasId: "TSTALIASID", localeId: "en_US",
  sessionId: "user-42",
  text: "Where is my order 12345?",
}));
console.log(res.messages, res.sessionState);
```

## Pricing

- **Text request:** $0.00075 each.
- **Speech request:** $0.004 each.
- **Streaming:** $0.0065 per minute.

## Gotchas

- **Sample utterances quality matters.** Provide 15-50 varied examples per intent.
- **Slot validation in Lambda** — Lex sends a `DialogCodeHook` event; you can correct/reprompt.
- **Voice channels (Connect)** require careful prompt tuning for naturalness.
- **V1 → V2 migration** is non-trivial. Build new bots in V2.

## Related

- [Bedrock](./bedrock.md) — for open-domain LLM-driven bots
- [Amazon Connect](#) — call center platform that uses Lex
- [Lambda](../01-compute/lambda.md) — fulfillment
