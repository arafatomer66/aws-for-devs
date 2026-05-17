# Polly — Text to Speech

**TL;DR** — Convert text → MP3/OGG/PCM. Many voices, languages, neural quality. Pay per character.

## What it is

A text-to-speech service. Standard voices + Neural voices + Long-form voices (for podcasts/audiobooks) + Generative voices.

## Usage

```js
import { PollyClient, SynthesizeSpeechCommand } from "@aws-sdk/client-polly";
import { writeFile } from "fs/promises";
const polly = new PollyClient({ region: "ap-south-1" });

const res = await polly.send(new SynthesizeSpeechCommand({
  Text: "Hello, your order has been placed.",
  VoiceId: "Joanna",           // see ListVoices
  Engine: "neural",
  OutputFormat: "mp3",
}));
await writeFile("out.mp3", Buffer.from(await res.AudioStream.transformToByteArray()));
```

### SSML for control

```xml
<speak>
  Welcome to <emphasis level="strong">ShareDeal</emphasis>.
  <break time="500ms"/> Today's deals are
  <prosody rate="slow">amazing</prosody>.
</speak>
```

## Pricing

- **Standard:** $4.00 per 1M chars.
- **Neural:** $16.00 per 1M chars.
- **Long-form:** $100 per 1M chars.
- **Generative:** $30 per 1M chars (premium quality).

## Real-world example

> IVR / phone agent: user hears "Press 1 for orders…" via Polly. Voice cached in S3 to avoid re-synthesizing static prompts.

## Gotchas

- **Caching matters.** Static prompts → synthesize once, store in S3.
- **Some voices/languages only available in some regions.**
- **3000-character cap per request** for synchronous synthesis; async for longer.
- **Generative voices need lexicon licensing** in some commercial use cases — read the terms.

## Related

- [Transcribe](./transcribe.md)
- [Translate](./translate.md)
