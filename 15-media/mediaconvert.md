# MediaConvert

**TL;DR** — Batch video transcoding. Take a source video, output 1080p MP4, 720p HLS, 480p DASH, thumbnails, captions, in parallel. Pay per output minute.

## What it is

A file-based video processing service. Submit a job describing input(s) + output(s); MediaConvert renders, transcodes, packages, and writes outputs to S3.

## Key concepts

- **Job** — one request with inputs + output groups.
- **Output group** — a packaging type (File / HLS / DASH / Microsoft Smooth / CMAF).
- **Queues** — on-demand or reserved.
- **Job templates** — reusable presets.
- **Acceleration** — turbo mode for faster transcodes at higher cost.

## Real-world example

> User-uploaded videos in a social app:
> - S3 upload event → Lambda → MediaConvert job.
> - Output 1: HLS adaptive bitrate (240p / 480p / 720p / 1080p).
> - Output 2: thumbnail at 5s mark.
> - Outputs land in `s3://cdn-origin/videos/{id}/`.
> - CloudFront serves.

## Usage

```js
import { MediaConvertClient, CreateJobCommand } from "@aws-sdk/client-mediaconvert";
const mc = new MediaConvertClient({ region: "ap-south-1", endpoint: "<account-specific-endpoint>" });

await mc.send(new CreateJobCommand({
  Role: "arn:aws:iam::..:role/MediaConvertRole",
  Settings: {
    Inputs: [{ FileInput: "s3://uploads/raw/video1.mov" }],
    OutputGroups: [{
      Name: "HLS",
      OutputGroupSettings: { Type: "HLS_GROUP_SETTINGS", HlsGroupSettings: { Destination: "s3://cdn-origin/videos/v1/" } },
      Outputs: [
        { Preset: "System-Avc_16x9_720p_29_97fps_3500kbps" },
        { Preset: "System-Avc_16x9_480p_29_97fps_1500kbps" },
      ],
    }],
  },
}));
```

Get your account-specific endpoint:
```bash
aws mediaconvert describe-endpoints
```

## Pricing

- ~$0.0075-$0.075 per minute of output depending on resolution + codec + acceleration.
- AV1 / 4K cost more than 720p H.264.

## Gotchas

- **Endpoint is per-account** — must call DescribeEndpoints first.
- **HLS / DASH packaging** needs careful playlist setup for adaptive bitrate.
- **DRM** supported (Widevine, FairPlay, PlayReady) but adds setup.
- **Queue contention** — on-demand queues throttle under huge load; reserve for predictable workloads.

## Related

- [MediaLive](./medialive.md) — live streaming, not batch
- [S3](../02-storage/s3.md)
- [CloudFront](../04-networking/cloudfront.md) — deliver outputs
- [Lambda](../01-compute/lambda.md) — trigger jobs
