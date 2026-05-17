# ACM — AWS Certificate Manager

**TL;DR** — Free public TLS certs for AWS services (CloudFront, ALB, API Gateway). Auto-renewal. Private CA available for internal certs (paid).

## What it is

A managed cert authority + cert distributor for AWS services. Public certs are **free** (DV — Domain Validation). Private CA (for internal mTLS / VPN) is paid.

## Key concepts

- **Public certificate** — for *.example.com, used on internet-facing endpoints. Free.
- **Private certificate** — issued by your ACM Private CA. Paid.
- **Validation methods:** DNS (preferred, auto-renews) or Email.
- **Renewal:** automatic if DNS validation; AWS rotates the cert.
- **Wildcard supported** — `*.example.com`.
- **SAN** — multiple names in one cert.
- **Regional:** ACM certs are regional. Same cert in multiple regions → request once per region.

## Real-world example

> ShareDeal needs HTTPS:
> - Request a wildcard cert `*.selefe.com` in `us-east-1` (for CloudFront).
> - Request another cert for `*.selefe.com` in `ap-south-1` (for ALB).
> - Use DNS validation; add CNAMEs to Route 53.
> - AWS renews automatically before expiry.

## Usage

### Request a public cert

```bash
aws acm request-certificate \
  --domain-name "*.selefe.com" \
  --subject-alternative-names "selefe.com" \
  --validation-method DNS
```

ACM returns a CNAME record to add to your DNS. If your domain is in Route 53:

```bash
aws acm describe-certificate --certificate-arn arn:... \
  --query 'Certificate.DomainValidationOptions[].ResourceRecord'
# Add the CNAME(s) to Route 53; ACM auto-validates once visible.
```

### Attach to ALB

```bash
aws elbv2 modify-listener --listener-arn arn:... --certificates CertificateArn=arn:aws:acm:...
```

### Attach to CloudFront

```bash
aws cloudfront update-distribution ... ViewerCertificate.ACMCertificateArn=arn:aws:acm:us-east-1:...
```

(CloudFront only accepts ACM certs from `us-east-1`.)

### CDK convenience

```ts
const cert = new acm.Certificate(this, "Cert", {
  domainName: "*.selefe.com",
  validation: acm.CertificateValidation.fromDns(hostedZone),
});
```

## Pricing

- **Public certs:** free.
- **Private CA:** $400/mo for the CA + $0.75 per cert issued (first 1000), then cheaper tiers.

## Gotchas

- **CloudFront wants `us-east-1` certs.** Other services use the cert in their region.
- **Email validation is fragile** — DNS validation is the better default.
- **Cert can't be exported** unless from Private CA (or imported). You can't grab the private key for an ACM-issued public cert.
- **EC2 needs to use the cert via ELB / CloudFront.** ACM doesn't deliver the private key to your EC2 (you'd need to use Private CA or Let's Encrypt for that).
- **65 SANs max per cert.**

## Related

- [CloudFront](../04-networking/cloudfront.md)
- [ALB](../04-networking/elb.md)
- [API Gateway](../04-networking/api-gateway.md)
- [Route 53](../04-networking/route53.md)
