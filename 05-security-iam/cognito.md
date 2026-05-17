# Cognito

**TL;DR** — End-user authentication for your apps. User Pools (signup/login/JWT) + Identity Pools (federated AWS credentials). Auth0/Firebase Auth alternative, AWS-native.

## What it is

Two distinct services bundled:

### 1. Cognito User Pool
A user directory. Handles signup, login, email/SMS verification, MFA, password reset, social federation (Google, Facebook, Apple, SAML, OIDC). Issues JWTs (access + ID + refresh tokens).

### 2. Cognito Identity Pool (Federated Identities)
Exchanges a token (from User Pool, Google, etc.) for **temporary AWS credentials**. Let an authenticated user `s3:PutObject` to their own folder, no backend in between.

Most apps use User Pool only. Identity Pool is for direct AWS access from mobile/web.

## Key concepts (User Pool)

- **User Pool** — your users.
- **App Client** — public (mobile/web) or confidential (server). Configure OAuth grants, callback URLs.
- **Hosted UI** — Cognito-hosted login page (customizable colors/logo).
- **Triggers** — Lambdas at lifecycle hooks (pre-sign-up, post-confirmation, pre-token-generation, etc.).
- **Groups** — assign users; embed in JWT claims for RBAC.
- **MFA** — TOTP, SMS, hardware (FIDO2/WebAuthn).
- **Custom attributes** — extra user fields.
- **Federation** — Google/Facebook/Apple/SAML/OIDC IdPs.

## Real-world example

> ShareDeal mobile app:
> - Cognito User Pool: phone-based signup (BD users), OTP verification.
> - App client (public): native mobile, PKCE flow.
> - Group `vip-customers` → JWT claim → API Gateway authorizer scopes them.
> - Pre-token Lambda adds `tenantId` claim from DynamoDB.

## Usage

### Create User Pool

Console-driven mostly, but CLI:
```bash
aws cognito-idp create-user-pool \
  --pool-name sd-users \
  --policies 'PasswordPolicy={MinimumLength=10,RequireUppercase=true,RequireNumbers=true,RequireSymbols=true}' \
  --auto-verified-attributes email \
  --mfa-configuration OPTIONAL --enabled-mfas SOFTWARE_TOKEN_MFA
```

### Create app client

```bash
aws cognito-idp create-user-pool-client \
  --user-pool-id ap-south-1_XXXX \
  --client-name web-app \
  --no-generate-secret \
  --explicit-auth-flows ALLOW_USER_SRP_AUTH ALLOW_REFRESH_TOKEN_AUTH \
  --callback-urls https://app.selefe.com/callback \
  --supported-identity-providers COGNITO Google \
  --allowed-o-auth-flows code --allowed-o-auth-scopes openid email profile \
  --allowed-o-auth-flows-user-pool-client
```

### Sign up + confirm + sign in (CLI, for testing)

```bash
aws cognito-idp sign-up --client-id <app-client-id> \
  --username arif@example.com --password 'Hunter22!' \
  --user-attributes Name=email,Value=arif@example.com

# (User receives a code by email)
aws cognito-idp confirm-sign-up --client-id <app-client-id> \
  --username arif@example.com --confirmation-code 123456

aws cognito-idp initiate-auth --client-id <app-client-id> \
  --auth-flow USER_PASSWORD_AUTH \
  --auth-parameters USERNAME=arif@example.com,PASSWORD='Hunter22!'
# → IdToken, AccessToken, RefreshToken
```

### Use the JWT

API Gateway HTTP API authorizer:
```yaml
JwtConfiguration:
  Issuer: https://cognito-idp.ap-south-1.amazonaws.com/<user-pool-id>
  Audience: [<app-client-id>]
```

Or in app code, verify the JWT against the pool's JWKS.

### Hosted UI

`https://<your-domain>.auth.ap-south-1.amazoncognito.com/login?response_type=code&client_id=...&redirect_uri=...`

Easiest path to a working login.

## Pricing

- **Free tier:** 50,000 MAUs (monthly active users) free.
- **Above:** ~$0.0055 per MAU. Plus higher tiers for advanced security features.

Cheap for most apps. Compare with Auth0 / Firebase pricing.

## Identity Pool quick example

Exchange a token for AWS creds:
```js
import { CognitoIdentityClient } from "@aws-sdk/client-cognito-identity";
import { fromCognitoIdentityPool } from "@aws-sdk/credential-providers";

const creds = fromCognitoIdentityPool({
  client: new CognitoIdentityClient({ region: "ap-south-1" }),
  identityPoolId: "ap-south-1:xxxxxxxx",
  logins: { "cognito-idp.ap-south-1.amazonaws.com/<user-pool-id>": idToken },
});
// Now creds can sign AWS API calls scoped by an IAM role attached to the identity pool
```

## Gotchas

- **User Pool is region-pinned** — can't move; replicate manually for DR.
- **Quotas** — sign-in rate limits per IP and per pool. Plan for legit traffic spikes.
- **Custom attributes are immutable** once created (in terms of name/type).
- **Hosted UI is dated-looking** — most teams build their own login screen.
- **Lambda triggers can be invoked at high concurrency** — keep them fast.
- **Migration from another identity provider** can be tricky — pre-sign-up trigger handles "migrating user."
- **Token expiry** — ID/Access tokens 1 hr default; refresh up to 10 years.

## Related

- [API Gateway](../04-networking/api-gateway.md) — Cognito-authorized routes
- [IAM](./iam.md) — for *your* internal users (Identity Center), not end users
- [Amplify](../13-developer-tools/amplify.md) — wraps Cognito for mobile/web
