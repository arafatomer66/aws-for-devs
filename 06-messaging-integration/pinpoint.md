# Pinpoint

**TL;DR** — Multi-channel customer engagement: push notifications (APNs/FCM), SMS, email, voice, in-app. Segments, journeys, analytics. Use for marketing + lifecycle messaging; SES for raw transactional email.

## What it is

A campaign + messaging service. Maintain an **endpoint** per user device/contact, segment them by attributes, run campaigns and **journeys** (multi-step flows), measure open/click/conversion.

In 2024+ AWS started splitting Pinpoint into **End User Messaging** sub-services (SMS, Voice, Push API) for raw send and **Pinpoint Engage** for the marketing layer. Naming is evolving — check the console.

## Channels

- **Push** — iOS APNs, Android FCM, ADM (Amazon), Baidu.
- **SMS** — short codes / long codes / sender IDs; varies by country.
- **Voice** — TTS phone calls.
- **Email** — uses SES under the hood.
- **In-app** — Amplify/SDK shows messages inside your app.
- **Custom** — Lambda channel for anything else.

## Key concepts

- **Project** — top-level resource.
- **Endpoint** — one delivery address (device token, phone, email) for one user.
- **User** — owns multiple endpoints.
- **Segment** — filtered set of endpoints (e.g., "BD users, last_active < 7d").
- **Campaign** — one-shot message to a segment.
- **Journey** — multi-step flow (e.g., send push → wait 24h → if not opened, SMS).
- **Event** — your app reports custom events (signup, purchase) for targeting.

## Real-world example

> ShareDeal mobile app retention:
> - Endpoint registered per install (FCM token).
> - Segment: `daysSinceLastOpen >= 7 AND country == BD`.
> - Journey: day 7 → push, day 9 → SMS with promo, day 14 → email.
> - 22% reactivation rate; tracked in Pinpoint analytics.

## Usage

### Register endpoints (Amplify Push)

Amplify SDK auto-registers device tokens with Pinpoint. Or:

```js
import { PinpointClient, UpdateEndpointCommand } from "@aws-sdk/client-pinpoint";
const pp = new PinpointClient({ region: "ap-south-1" });
await pp.send(new UpdateEndpointCommand({
  ApplicationId: "<project-id>",
  EndpointId: "device-uuid-or-stable-id",
  EndpointRequest: {
    ChannelType: "GCM",
    Address: "fcm-token-here",
    User: { UserId: "u_42", UserAttributes: { Country: ["BD"] } },
    Attributes: { LastOpenDate: ["2026-05-10"] },
  },
}));
```

### Send a direct push

```js
import { SendMessagesCommand } from "@aws-sdk/client-pinpoint";
await pp.send(new SendMessagesCommand({
  ApplicationId: "<project-id>",
  MessageRequest: {
    Addresses: { "<endpoint-id>": { ChannelType: "GCM" } },
    MessageConfiguration: {
      GCMMessage: { Title: "ShareDeal", Body: "Today's offers are live!", Action: "OPEN_APP" },
    },
  },
}));
```

### Send SMS (Pinpoint SMS Voice v2 API)

```js
import { PinpointSMSVoiceV2Client, SendTextMessageCommand } from "@aws-sdk/client-pinpoint-sms-voice-v2";
const sms = new PinpointSMSVoiceV2Client({ region: "ap-south-1" });
await sms.send(new SendTextMessageCommand({
  DestinationPhoneNumber: "+8801xxxxxxxxx",
  MessageBody: "Your OTP is 123456",
  MessageType: "TRANSACTIONAL",
  OriginationIdentity: "<your-sender-id-or-phone-id>",
}));
```

## Pricing

- **Per channel:** push ~$1 per million; SMS varies by country (BD ~$0.012 per message); email cheap; voice ~$0.013/min.
- **Per active endpoint targeted:** $0.0012.
- **Per analytics event:** $0.000001.

Reasonable for most teams; SMS in some countries is the expensive line item.

## Pinpoint vs SNS vs SES

- **SNS** — raw pub/sub; you handle delivery logic. Good for transactional one-shots.
- **SES** — email-only, cheap and high-deliverability for transactional.
- **Pinpoint** — multi-channel campaigns, journeys, segments, analytics. Marketing/engagement.

## Gotchas

- **SMS regulations vary wildly.** Bangladesh requires registered sender IDs; US requires 10DLC; some countries are heavily restricted.
- **Push tokens expire.** Refresh endpoints on each app open.
- **Naming/transition.** AWS is migrating Pinpoint pieces to "AWS End User Messaging" branding — check current docs for any subservice.
- **Endpoint deduplication** — design `EndpointId` so the same device doesn't accumulate multiple records.

## Related

- [SES](./ses.md)
- [SNS](./sns.md)
- [Amplify](../13-developer-tools/amplify.md) — easiest mobile SDK integration
- [Cognito](../05-security-iam/cognito.md)
