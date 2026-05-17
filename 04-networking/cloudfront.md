# CloudFront

**TL;DR** — AWS's CDN. Caches your content at 600+ edge locations globally. Free TLS, free egress to AWS origins. Use it in front of S3, ALB, API Gateway, anything HTTP.

## What it is

Distributed cache + reverse proxy at edge. Users hit a nearby edge PoP; cache miss → fetch from your origin; cache hit → instant. Lowers latency, lowers egress cost, absorbs traffic spikes.

## Why it exists

- Latency: speed-of-light to far-away origins is slow.
- Bandwidth: every byte from EC2/S3 costs $0.09/GB egress; CloudFront egress is cheaper and free into many AWS origins.
- Resilience: CloudFront eats DDoS waves for you.
- TLS: free public certs via ACM.

## Key concepts

- **Distribution** — your CDN config. Has a domain like `d111111abcdef8.cloudfront.net`.
- **Origin** — backend (S3 bucket, ALB, custom HTTP origin, MediaStore, etc.).
- **Behavior** — match a path pattern → settings (cache TTLs, headers forwarded, viewer protocol).
- **Cache policy** — what makes the cache key (path? query string? specific headers?).
- **Origin request policy** — what gets forwarded to the origin on miss.
- **Response headers policy** — what to add to responses (CORS, security headers).
- **OAC (Origin Access Control)** — auth between CloudFront and S3 (replaces the old OAI).
- **Functions / Lambda@Edge** — run code at the edge.
- **Signed URLs / Signed cookies** — restrict access (paid content, time-limited).
- **Geo restrictions** — whitelist/blacklist countries.
- **WAF** — attach AWS WAF to a distribution.

## Real-world example

> ShareDeal serves images via CloudFront:
> - Origin: S3 bucket `selefe-uploads` (private, OAC).
> - Behavior `/*.jpg, *.png, *.webp`: cache 1 day, brotli compression.
> - Behavior `/api/*`: forwards to ALB origin, **caching disabled** for dynamic content.
> - Custom domain `cdn.selefe.com` via Route 53 alias + ACM cert.

## Usage

### CDK example

```ts
const distribution = new cloudfront.Distribution(this, "Cdn", {
  defaultBehavior: {
    origin: new origins.S3Origin(bucket, { originAccessIdentity: ... }),
    viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
    cachePolicy: cloudfront.CachePolicy.CACHING_OPTIMIZED,
    compress: true,
  },
  additionalBehaviors: {
    "/api/*": {
      origin: new origins.HttpOrigin(albDomain),
      cachePolicy: cloudfront.CachePolicy.CACHING_DISABLED,
      originRequestPolicy: cloudfront.OriginRequestPolicy.ALL_VIEWER,
      viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.HTTPS_ONLY,
    },
  },
  certificate: acmCert,
  domainNames: ["cdn.selefe.com"],
});
```

### Invalidate cache

```bash
aws cloudfront create-invalidation \
  --distribution-id E1A2B3C4D5 \
  --paths "/index.html" "/static/*"
```

First 1000 invalidation paths free per month, $0.005 each after.

### Signed URL (Node SDK v3)

```js
import { getSignedUrl } from "@aws-sdk/cloudfront-signer";
const url = getSignedUrl({
  url: "https://cdn.selefe.com/private/file.pdf",
  keyPairId: "K1ABCD...",
  privateKey: fs.readFileSync("./cf-private.pem", "utf-8"),
  dateLessThan: new Date(Date.now() + 5 * 60 * 1000),
});
```

## CloudFront Functions vs Lambda@Edge

| | CloudFront Functions | Lambda@Edge |
|---|---|---|
| Runtime | JS ES5-like, ~1 ms max | Node/Python, full lang |
| Use | Header tweaks, URL rewrites | Heavy logic, auth, A/B tests |
| Latency | Sub-ms | A few ms |
| Cost | $0.10 per million | Higher |

## Pricing

- **Free tier:** 1 TB/mo + 10M requests/mo.
- **Egress to internet:** ~$0.085/GB in NA/EU, more elsewhere. Cheaper than direct S3 egress.
- **AWS → CloudFront origin pulls:** free in most cases.
- **Requests:** $0.0075-$0.01 per 10k.

## Gotchas

- **Cache key default** doesn't include query strings or most headers. Customize via cache policy if you need.
- **Origin failover** works but is slow to detect — couple seconds.
- **Custom domain** needs ACM cert **in `us-east-1`** (CloudFront is global, ACM is regional, distributions only use us-east-1 ACM).
- **Origin shield** is an extra caching layer in front of your origin. Pays off for high-fanout content.
- **GET-only by default for caching.** POST/PUT bypass cache.
- **Brotli + gzip both supported** — let CloudFront pick.
- **Invalidations cost money** — better to use versioned filenames (`/static/v123.js`) and never invalidate.

## Related

- [S3](../02-storage/s3.md)
- [Route 53](./route53.md)
- [ACM](../05-security-iam/acm.md)
- [WAF](../05-security-iam/waf.md)
- [Global Accelerator](./global-accelerator.md) — different tool, anycast TCP/UDP
