# SES — Simple Email Service

**TL;DR** — Send + receive email at internet scale. Transactional (signup, password reset, receipts) and marketing. Cheap ($0.10 per 1k emails). Reputation management built in.

## What it is

A managed SMTP + API for email. Two directions:
- **Outbound** — your app sends emails (via SDK or SMTP).
- **Inbound** — receive emails to S3 / Lambda / SNS (rare but useful for support@-style inboxes).

## Key concepts

- **Identity** — verified email address or domain you're allowed to send from.
- **Domain verification** — DKIM, SPF, DMARC DNS records.
- **Sandbox vs Production** — new accounts start in sandbox (200 emails/day, only to verified addresses). Request prod access via support ticket.
- **Configuration set** — group of rules (event tracking, IP pool, custom headers).
- **Event destinations** — bounces/complaints/opens/clicks → CloudWatch / SNS / Firehose / EventBridge.
- **Suppression list** — auto-suppress bounced/complained addresses.
- **Templates** — reusable HTML/text with substitutions.

## Real-world example

> ShareDeal sends order receipts:
> - Domain `selefe.com` verified with DKIM/SPF/DMARC.
> - Lambda fires `SendEmailCommand` with template `OrderReceipt`.
> - Bounces/complaints route via SNS → DynamoDB → suppression check.
> - SES Reputation Dashboard tracks complaint rate; alarms at > 0.1%.

## Usage

### Verify a domain

```bash
aws sesv2 create-email-identity --email-identity selefe.com
# Returns DNS records to add (DKIM CNAMEs); add them to Route 53
```

Wait for `VerifiedForSendingStatus = true`.

### Send via SDK (Node v3)

```js
import { SESv2Client, SendEmailCommand } from "@aws-sdk/client-sesv2";
const ses = new SESv2Client({ region: "ap-south-1" });

await ses.send(new SendEmailCommand({
  FromEmailAddress: "noreply@selefe.com",
  Destination: { ToAddresses: ["user@example.com"] },
  Content: {
    Simple: {
      Subject: { Data: "Your order is confirmed" },
      Body: {
        Html: { Data: "<h1>Thanks!</h1><p>Order #ord_42</p>" },
        Text: { Data: "Thanks! Order #ord_42" },
      },
    },
  },
  ConfigurationSetName: "transactional",
}));
```

### Send templated

```js
import { SendBulkEmailCommand } from "@aws-sdk/client-sesv2";
await ses.send(new SendBulkEmailCommand({
  FromEmailAddress: "noreply@selefe.com",
  DefaultContent: { Template: { TemplateName: "OrderReceipt", TemplateData: "{}" } },
  BulkEmailEntries: [
    { Destination: { ToAddresses: ["a@x.com"] }, ReplacementEmailContent: { ReplacementTemplate: { ReplacementTemplateData: JSON.stringify({ order: "ord_42", total: 1200 }) } } },
  ],
}));
```

### SMTP

Generate SMTP creds from console → use any SMTP client. Endpoint: `email-smtp.ap-south-1.amazonaws.com:587` (STARTTLS).

## Pricing

- **Outbound:** $0.10 per 1,000 emails (from EC2/Lambda; cheaper than third parties).
- **Attachments:** $0.12/GB.
- **Inbound:** $0.10 per 1,000 + $0.09/GB.
- **First 62,000 emails/mo free from EC2/Lambda** (always free tier).

## Gotchas

- **Sandbox by default** — request production access early or you can't send to real users.
- **Reputation matters.** Bounce rate > 5% or complaint rate > 0.1% → AWS pauses sending.
- **DKIM + SPF + DMARC** — set all three. Gmail/Yahoo now reject misconfigured senders.
- **SES region is independent.** You can send from `us-east-1` even if app is in `ap-south-1`. Pick a region with mature reputation.
- **List-Unsubscribe header** is required for marketing email — set it.
- **Suppression list is per-account, all regions** — useful.

## SES vs Pinpoint vs WorkMail vs SendGrid

| | SES | Pinpoint | SendGrid/Postmark |
|---|---|---|---|
| Transactional | ✅ | ✅ | ✅ |
| Marketing campaigns | Limited | ✅ | ✅ |
| Per-recipient analytics | Limited | ✅ | ✅ |
| Cost | Cheapest | Mid | Higher |

For pure transactional → SES.
For marketing/journeys → Pinpoint or SendGrid/Mailchimp.

## Related

- [Pinpoint](./pinpoint.md)
- [SNS](./sns.md) — receive SES events
- [Lambda](../01-compute/lambda.md)
- [Route 53](../04-networking/route53.md) — DKIM/SPF DNS
