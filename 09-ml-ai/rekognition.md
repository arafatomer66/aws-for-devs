# Rekognition

**TL;DR** — Image + video analysis API. Detect labels, faces, text, celebrities, unsafe content, PPE, custom labels. Pay per image/minute.

## What it can do

- **Labels** — "dog", "tree", "car" in an image.
- **Faces** — detect, compare, search a collection.
- **Text in image (OCR)** — but use **Textract** for documents.
- **Moderation** — explicit, suggestive, violence detection.
- **Custom Labels** — train your own classifier with a few hundred images.
- **PPE detection** — hardhats, masks.
- **Video analysis** — labels, people, faces across frames.
- **Streaming Video Events** — react to live video.

## Real-world example

> User-uploaded photos in an app:
> - On upload → Lambda → Rekognition `DetectModerationLabels`.
> - If `Explicit Nudity` or `Violence` score > 80, reject the upload.
> - For e-commerce: `DetectLabels` auto-tags product photos.

## Usage

```js
import { RekognitionClient, DetectLabelsCommand } from "@aws-sdk/client-rekognition";
const rek = new RekognitionClient({ region: "ap-south-1" });

const res = await rek.send(new DetectLabelsCommand({
  Image: { S3Object: { Bucket: "uploads", Name: "img.jpg" } },
  MaxLabels: 10, MinConfidence: 80,
}));
console.log(res.Labels);
```

## Pricing

- **Images:** $1 per 1,000 (first 1M); cheaper above.
- **Video:** $0.10 per minute analyzed.
- **Face collections:** $0.01 per face-mo stored.

## Gotchas

- **Faces ≠ identity.** Don't deploy face recognition for surveillance/identity-critical use without strong governance.
- **Confidence threshold** matters — false positives at low thresholds.
- **Video analysis is async** — submit job, poll, fetch results.
- **No live webcam streaming** API — use Kinesis Video Streams + Stream Events.

## Related

- [Textract](./textract.md)
- [Lambda](../01-compute/lambda.md)
