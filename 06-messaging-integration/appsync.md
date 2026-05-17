# AppSync — Managed GraphQL

**TL;DR** — Managed GraphQL backend. Define schema, attach resolvers to DynamoDB / RDS / Lambda / HTTP / OpenSearch. Real-time subscriptions over WebSocket. AppSync Events (newer) for pub/sub without GraphQL.

## What it is

A GraphQL gateway: you define a schema, AppSync runs queries/mutations/subscriptions against backing data sources, handles auth, real-time push, caching.

## Key concepts

- **API** — your AppSync GraphQL API (or AppSync Events API).
- **Schema** — GraphQL SDL.
- **Resolver** — code (VTL — older JS Pipeline Resolvers — newer) that maps a field to a data source.
- **Data source** — DynamoDB / RDS Proxy / Lambda / HTTP / OpenSearch / EventBridge / None.
- **Auth modes:** API key, Cognito User Pool, IAM, OIDC, Lambda.
- **Real-time subscriptions** — WebSocket-based push when mutations match a filter.
- **Caching** — built-in TTL cache.
- **Merged APIs** — federate multiple AppSync APIs (think Apollo Federation).
- **AppSync Events** — pub/sub channels without GraphQL — for WebSocket pub/sub.

## Real-world example

> Real-time chat app:
> - Schema: `Message { id, room, author, text }`, mutation `sendMessage`, subscription `onMessageSent(room)`.
> - DynamoDB data source.
> - Client subscribes; AppSync pushes new messages over WebSocket.
> - No Lambda or backend code; just resolvers.

## Usage

### Schema (SDL)

```graphql
type Message {
  id: ID!
  room: String!
  author: String!
  text: String!
  createdAt: AWSDateTime
}

type Query {
  messages(room: String!, limit: Int): [Message]
}

type Mutation {
  sendMessage(room: String!, author: String!, text: String!): Message
}

type Subscription {
  onMessageSent(room: String!): Message
    @aws_subscribe(mutations: ["sendMessage"])
}
```

### JS Pipeline Resolver (DynamoDB PutItem)

```js
import { util } from "@aws-appsync/utils";

export function request(ctx) {
  return {
    operation: "PutItem",
    key: util.dynamodb.toMapValues({ id: util.autoId() }),
    attributeValues: util.dynamodb.toMapValues({
      room: ctx.args.room,
      author: ctx.args.author,
      text: ctx.args.text,
      createdAt: util.time.nowISO8601(),
    }),
  };
}
export function response(ctx) { return ctx.result; }
```

### Pricing

- **$4.00 per million queries/mutations.**
- **$2.00 per million real-time updates.**
- **Connection-minutes:** $0.08 per million.
- **Cache:** instance-hour like ElastiCache.

## Gotchas

- **Schema design matters more than for REST** — N+1 queries can balloon.
- **VTL is awkward** — use JS Pipeline Resolvers for new APIs.
- **Subscriptions filter on the server side** — overly broad subscriptions get expensive.
- **No automatic GraphQL playground in prod** — bring your own (GraphiQL, etc.).
- **AppSync Events API** is the new way to do simple WebSocket pub/sub without GraphQL.

## Related

- [DynamoDB](../03-database/dynamodb.md)
- [Cognito](../05-security-iam/cognito.md)
- [API Gateway](../04-networking/api-gateway.md) — for REST/HTTP
- [Amplify](../13-developer-tools/amplify.md) — wraps AppSync for mobile/web
