# Amazon Chime SDK

**TL;DR** — Embed audio/video calls + screen share + messaging into your app. WebRTC under the hood, AWS handles SFU + media servers. Pay per attendee-minute.

## What it is

A set of SDKs (JS, iOS, Android, React Native, C++) and AWS-side services for building real-time comms — telehealth visits, customer support video, classrooms, virtual events. **Not the same as Amazon Chime the consumer product** (which has been retired).

## Key concepts

- **Meeting** — a call/session.
- **Attendee** — one participant.
- **Media Region** — where the SFU runs.
- **Messaging** — separate Chime SDK Messaging service for chat channels.
- **Voice Connector** — SIP trunking for telephony.
- **Media pipelines** — server-side recording, transcription, broadcast.

## Real-world example

> Telemedicine app:
> - Doctor books slot → app creates a Chime meeting → both attendees join.
> - JavaScript SDK renders the video grid in-browser.
> - Media pipeline records the visit + Transcribe generates a transcript for the EMR.

## Usage (Node — server creates meeting)

```js
import { ChimeSDKMeetingsClient, CreateMeetingCommand, CreateAttendeeCommand } from "@aws-sdk/client-chime-sdk-meetings";
const chime = new ChimeSDKMeetingsClient({ region: "ap-south-1" });

const { Meeting } = await chime.send(new CreateMeetingCommand({
  ClientRequestToken: "uuid-here",
  MediaRegion: "ap-south-1",
  ExternalMeetingId: "visit-42",
}));

const { Attendee } = await chime.send(new CreateAttendeeCommand({
  MeetingId: Meeting.MeetingId,
  ExternalUserId: "doctor-1",
}));

// Return Meeting + Attendee to the client; client SDK uses them to join.
```

Client-side joining via `amazon-chime-sdk-js`.

## Pricing

- **Audio:** $0.0017 per attendee-minute.
- **Video:** $0.0017 per attendee-minute.
- **Screen share:** $0.0017 per attendee-minute.
- **Messaging:** $0.0002 per message.
- **Recording / Transcription / Voice Connector:** separate per-minute fees.

A 10-person 1-hour video meeting = ~$1.

## Gotchas

- **MediaRegion latency** matters — set close to participants.
- **Browser audio echo cancellation** varies — use the SDK's processors.
- **Recording requires media pipelines** + S3 destination + proper IAM.
- **Chime SDK Messaging** is separate from MediaLive's pipelines.
- **TURN servers / firewalls** — corporate networks sometimes block; the SDK handles fallback.

## Chime SDK vs alternatives

- **Twilio Video / Daily.co / Agora** — easier DX, broader ecosystems.
- **Chime SDK** — cheapest at scale, AWS-native, broader stack control.

## Related

- [Transcribe](../09-ml-ai/transcribe.md) — call transcription
- [Translate](../09-ml-ai/translate.md) — real-time captions
- [S3](../02-storage/s3.md) — recordings
