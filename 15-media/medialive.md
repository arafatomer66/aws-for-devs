# MediaLive

**TL;DR** — Live video encoding. Take RTMP/RTP/HLS/SRT input, output HLS/DASH/MediaStore. For live streaming events at broadcast quality.

## What it is

A live video processing service. Pairs with **MediaPackage** (just-in-time packaging + origin) and **MediaStore / S3** (storage) for an end-to-end live workflow.

## Companion services

- **MediaLive** — encode/transcode live streams.
- **MediaPackage** — package + DVR window + DRM.
- **MediaStore** — low-latency origin store (being de-emphasized in favor of S3 + CloudFront).
- **MediaConnect** — secure live-feed transport between facilities.
- **MediaTailor** — server-side ad insertion (SSAI).

## Key concepts

- **Channel** — a live encoding pipeline.
- **Input** — RTMP push / pull, RTP, MP4 file loop, etc.
- **Output groups** — HLS / DASH / MediaPackage / Archive (S3).
- **Reservations** — discounted committed capacity.

## Real-world example

> Live concert stream:
> - Encoder on-site pushes RTMP to MediaLive input.
> - MediaLive transcodes to ABR (240p–1080p).
> - Output to MediaPackage → CloudFront → viewers.
> - DVR window 6 hours (rewind).

## Pricing

- Per-channel-hour: depends on input + output resolution.
- 1080p input → 4 outputs (1080/720/480/240) ≈ $3-5/hr.
- Reserved 1-year commits cut ~30-50%.

## Gotchas

- **Channels left running cost money 24/7.** Stop them between events.
- **Audio/video alignment** is the source of most "looks broken" bugs.
- **Codec licensing** for HEVC, AV1 can require additional setup.
- **End-to-end testing** before a live event is essential.

## Related

- [MediaConvert](./mediaconvert.md)
- [CloudFront](../04-networking/cloudfront.md)
- [S3](../02-storage/s3.md)
