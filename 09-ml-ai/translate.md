# Translate

**TL;DR** — Neural machine translation. 75+ languages. Sync and batch. Custom terminology to enforce brand/product names.

## Usage

```js
import { TranslateClient, TranslateTextCommand } from "@aws-sdk/client-translate";
const t = new TranslateClient({ region: "ap-south-1" });

const res = await t.send(new TranslateTextCommand({
  Text: "Hello, welcome to ShareDeal.",
  SourceLanguageCode: "en",
  TargetLanguageCode: "bn",
}));
console.log(res.TranslatedText);
```

### Custom terminology

Upload a CSV mapping (`source,target`) so "ShareDeal" stays "ShareDeal" in every language. Pass `TerminologyNames: ["sd-brand"]` in requests.

## Pricing

- **$15 per million characters** (real-time and batch standard).
- **Active Custom Translation** higher.

## Real-world example

> ShareDeal app supports English + Bengali. Product descriptions in English are auto-translated to Bengali nightly via batch translation. Custom terminology ensures brand names stay intact.

## Gotchas

- **Language pairs matter** — quality varies; major pairs (en↔es, en↔fr) excellent, niche pairs less so.
- **Costs scale by chars including spaces & punctuation.**
- **5000 byte max per real-time request**; batch handles bigger.

## Related

- [Comprehend](./comprehend.md)
- [Polly](./polly.md)
