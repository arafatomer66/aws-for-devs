# API Gateway

**TL;DR** — Managed HTTP/REST/WebSocket APIs. Authenticates, throttles, transforms, routes to Lambda / ECS / anything HTTP. Pay per request.

## What it is

A front door for your APIs. Define resources/methods, attach integrations (mostly Lambda), and API Gateway handles TLS, auth, rate limiting, request/response transformation, CORS.

## Three flavors

### 1. REST API (v1)
- Full-featured, oldest, most expensive.
- API keys, usage plans, request validators, models, deep transformations.
- Edge-optimized / Regional / Private endpoints.
- $3.50 per million requests.

### 2. HTTP API (v2)
- Newer, simpler, **cheaper (70% less)**, faster.
- JWT authorizers built in.
- Use this for new projects unless you specifically need REST API features.
- $1.00 per million requests.

### 3. WebSocket API
- Persistent connections.
- For chat, live dashboards, real-time games.
- $1.00 per million messages + $0.25 per million connection minutes.

## Key concepts

- **Stage** — deployment env (`prod`, `staging`).
- **Route / Resource** — URL pattern, e.g. `/users/{id}`.
- **Integration** — backend (Lambda, HTTP, AWS service direct).
- **Authorizer** — Cognito, JWT, Lambda (custom), IAM.
- **Usage Plan** — bind API keys to throttle/quota (REST API only).
- **Custom domain** — `api.myapp.com`.
- **Throttling** — per-API and per-key.
- **CORS** — first-class config.
- **WAF integration** — attach AWS WAF web ACL (REST API).

## Real-world example

> ShareDeal mobile API:
> - HTTP API v2, JWT authorizer (Cognito).
> - Routes:
>   - `GET /products/{id}` → Lambda
>   - `POST /orders` → Lambda
>   - `GET /orders` → Lambda
> - Custom domain `api.selefe.com`.
> - CloudWatch logs + metrics.

## Usage

### HTTP API + Lambda (CLI)

```bash
# Create HTTP API
aws apigatewayv2 create-api --name sd-api --protocol-type HTTP

# Integrate Lambda
aws apigatewayv2 create-integration \
  --api-id <api-id> --integration-type AWS_PROXY \
  --integration-uri arn:aws:lambda:ap-south-1:123:function:hello \
  --payload-format-version 2.0

# Route
aws apigatewayv2 create-route --api-id <api-id> \
  --route-key 'GET /hello' --target integrations/<int-id>

# Deploy
aws apigatewayv2 create-stage --api-id <api-id> --stage-name prod --auto-deploy
```

### SAM template (much easier for serverless)

```yaml
Resources:
  HelloFn:
    Type: AWS::Serverless::Function
    Properties:
      Runtime: nodejs20.x
      Handler: index.handler
      CodeUri: ./src/hello
      Events:
        Api:
          Type: HttpApi
          Properties:
            Path: /hello
            Method: get
```

### CDK example

```ts
const api = new apigwv2.HttpApi(this, "Api", {
  corsPreflight: { allowOrigins: ["*"], allowMethods: [apigwv2.CorsHttpMethod.ANY] },
});
api.addRoutes({
  path: "/hello",
  methods: [apigwv2.HttpMethod.GET],
  integration: new HttpLambdaIntegration("HelloInt", helloFn),
});
```

## Authorization patterns

- **Public** — no auth.
- **IAM** — SigV4 (server-to-server).
- **Cognito User Pool** — auto-validates JWT issued by your User Pool.
- **JWT authorizer** — any OIDC provider (Auth0, Okta, Firebase).
- **Lambda authorizer** — write your own (custom token format, opaque tokens).

## Custom domain

```bash
# 1. ACM cert (in same region for HTTP/REST regional, us-east-1 for REST edge-optimized)
# 2. Create custom domain
aws apigatewayv2 create-domain-name --domain-name api.selefe.com \
  --domain-name-configurations CertificateArn=arn:aws:acm:...

# 3. Map a stage
aws apigatewayv2 create-api-mapping --api-id <api-id> --domain-name api.selefe.com --stage prod

# 4. Route 53 alias to the custom domain's regional endpoint
```

## Pricing

- **HTTP API:** $1.00 per million requests + data transfer.
- **REST API:** $3.50 per million + caching surcharge if you turn on the regional cache.
- **WebSocket:** $1.00 per million messages + $0.25 per million connection-minutes.

## When NOT to use API Gateway

- High-throughput simple HTTP — use **ALB** with Lambda or ECS, often cheaper for steady high traffic.
- Lambda direct via **Function URL** — for super-simple public endpoints.
- gRPC / HTTP/2 streaming — use ALB or App Mesh.

## Gotchas

- **Hard 30-sec timeout** on REST/HTTP APIs. WebSocket can hold longer.
- **Payload limit 10 MB.**
- **Cold starts** — Lambda + API GW = full cold path 200-800ms first request.
- **REST API edge-optimized** uses CloudFront under the hood — TLS cert must be in `us-east-1`.
- **HTTP API doesn't support all REST API features** — usage plans, API keys, request validators, mock integrations. Migrate carefully.
- **WebSocket disconnect events aren't guaranteed** — clean up on a TTL.
- **CORS is at the API GW level for HTTP API**, must be done per-method for REST API.

## Related

- [Lambda](../01-compute/lambda.md)
- [Cognito](../05-security-iam/cognito.md)
- [ALB](./elb.md) — alternative front door
- [AppSync](../06-messaging-integration/appsync.md) — for GraphQL
