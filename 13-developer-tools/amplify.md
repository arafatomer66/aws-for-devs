# AWS Amplify

**TL;DR** — Full-stack dev platform for web + mobile. Hosting + auth + APIs + storage in one workflow. Like Vercel/Netlify with deeper AWS integration. Amplify Gen 2 (2024+) is much improved.

## What it is

Two distinct pieces:

### 1. Amplify Hosting
Deploy front-end web apps (React, Next.js, Angular, Vue, etc.) from a Git repo. Includes CDN, HTTPS, branch previews, PR previews. Closest to Vercel/Netlify.

### 2. Amplify Backend
A CDK-based set of opinionated constructs for app backends — auth (Cognito), data (AppSync/DynamoDB), storage (S3), functions (Lambda). You define resources in TypeScript and Amplify wires them up plus generates type-safe client SDKs.

**Amplify Gen 2** is a major rewrite using CDK + TypeScript first-class. Much better than Gen 1.

## Real-world example

> A Next.js app with auth + GraphQL API + file uploads:
>
> ```ts
> // amplify/backend.ts
> import { defineBackend } from "@aws-amplify/backend";
> import { defineAuth } from "@aws-amplify/backend-auth";
> import { defineData } from "@aws-amplify/backend-data";
> import { defineStorage } from "@aws-amplify/backend-storage";
>
> defineBackend({
>   auth: defineAuth({ loginWith: { email: true } }),
>   data: defineData({
>     schema: a.schema({
>       Todo: a.model({ title: a.string(), done: a.boolean() }).authorization(allow => [allow.owner()]),
>     }),
>   }),
>   storage: defineStorage({ name: "uploads", access: a => ({ "private/{entity_id}/*": [a.entity("identity").to(["read","write"])] }) }),
> });
> ```
>
> `npx ampx sandbox` runs a personal cloud env. `git push` deploys via Amplify Hosting.

## Usage

### Init + sandbox

```bash
npm create amplify
npx ampx sandbox    # personal dev env in your AWS account
```

### Client (React)

```tsx
import { generateClient } from "aws-amplify/data";
const client = generateClient<Schema>();
await client.models.Todo.create({ title: "Buy milk", done: false });
```

Fully type-safe based on the schema.

## Pricing

- **Hosting:** $0.01/min build + $0.023/GB stored + $0.15/GB served (cheaper at scale).
- **Backend pieces:** standard AWS (Cognito MAU, DynamoDB usage, etc.). No "Amplify tax."

A small app might cost $0-10/mo.

## Amplify vs Vercel/Netlify

- **Amplify** — deep AWS integration, owns your stack, more flexibility.
- **Vercel** — best DX for Next.js specifically; not in your AWS account.
- **Netlify** — similar to Vercel for general static + functions.

If you're already deep in AWS, Amplify is a great fit. Otherwise, Vercel often has better DX.

## Gotchas

- **Gen 1 vs Gen 2** are very different — make sure docs/examples match your version.
- **Amplify creates real AWS resources** — you can hand-tune them after.
- **Sandbox** = one personal env per developer. Watch costs across many devs.
- **Migration to plain CDK** is possible (Amplify is CDK under the hood) — easy escape hatch.

## Related

- [Cognito](../05-security-iam/cognito.md)
- [AppSync](../06-messaging-integration/appsync.md)
- [DynamoDB](../03-database/dynamodb.md)
- [CDK](../07-devops-iac/cdk.md)
