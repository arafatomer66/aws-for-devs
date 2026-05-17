# Transcribe — Speech to Text

**TL;DR** — Audio → text. Batch and streaming. Speaker diarization, custom vocab, PII redaction. Medical and Call Analytics variants.

## What it does

- Convert audio (mp3, wav, flac, etc.) → text.
- Batch (jobs) or real-time streaming.
- **Diarization** — who said what.
- **Custom vocabulary** — for product names, jargon.
- **PII redaction** — automatically replace SSN/CC with `***`.
- **Transcribe Medical** — HIPAA-eligible, medical terminology.
- **Call Analytics** — sentiment, talk time, interruptions, categories.

## Real-world example

> Customer-support call recordings:
> - Upload to S3 → Lambda starts Transcribe job → output JSON in S3.
> - Comprehend pulls topics + sentiment.
> - QuickSight dashboard: avg call length, top complaint topics, agent rankings.

## Usage

### Batch job

```bash
aws transcribe start-transcription-job \
  --transcription-job-name call-001 \
  --media MediaFileUri=s3://recordings/call-001.mp3 \
  --media-format mp3 \
  --language-code en-US \
  --output-bucket-name transcripts
```

### Streaming (Node)

```js
import { TranscribeStreamingClient, StartStreamTranscriptionCommand } from "@aws-sdk/client-transcribe-streaming";
const client = new TranscribeStreamingClient({ region: "ap-south-1" });

const command = new StartStreamTranscriptionCommand({
  LanguageCode: "en-US",
  MediaSampleRateHertz: 16000,
  MediaEncoding: "pcm",
  AudioStream: (async function* () {
    for await (const chunk of audioSource()) yield { AudioEvent: { AudioChunk: chunk } };
  })(),
});
const response = await client.send(command);
for await (const event of response.TranscriptResultStream) {
  if (event.TranscriptEvent) console.log(event.TranscriptEvent.Transcript.Results);
}
```

## Pricing

- **Batch:** $0.024 per minute (first 250k min/mo); cheaper above.
- **Streaming:** $0.024 per minute.
- **Call Analytics:** higher per-minute, post-call $0.03+/min.
- **Medical:** $0.075 per minute.

## Gotchas

- **Audio quality matters** — noisy recordings = poor transcripts.
- **Custom vocab is worth setting up** for domain-specific terms.
- **PII redaction is best-effort** — verify for compliance use cases.
- **Streaming sessions max 4 hours.**

## Related

- [Comprehend](./comprehend.md)
- [Polly](./polly.md)
- [Translate](./translate.md)
