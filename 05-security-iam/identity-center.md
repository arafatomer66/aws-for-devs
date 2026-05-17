# IAM Identity Center (formerly AWS SSO)

**TL;DR** — SSO for your team into multiple AWS accounts. Connect Okta/Google/Azure AD, define permission sets, users get a portal of accounts they can sign into.

## What it is

A central place to manage workforce identity across many AWS accounts. Replaces the bad old pattern of "make an IAM user in every account."

## Key concepts

- **Identity source** — Identity Center directory, AD Connector, or external IdP (SAML/SCIM).
- **User / Group** — synced from IdP.
- **Permission set** — a named bundle of IAM policies (e.g. `AdministratorAccess`, `Developer`).
- **Account assignment** — `(group, permission set, account)` triple.
- **Access portal** — `https://d-xxxx.awsapps.com/start` — users SSO here and pick an account/role.
- **AWS Access Portal CLI** — `aws sso login --profile dev` for command-line.

## Real-world example

> An org with 8 AWS accounts:
> - Identity Center connected to Google Workspace via SAML.
> - Groups: `engineers`, `data`, `sre`, `finance`.
> - Permission sets: `AdminAccess`, `DeveloperAccess`, `ReadOnly`, `BillingAccess`.
> - `engineers` group → `DeveloperAccess` in `dev` + `staging` accounts, `ReadOnly` in `prod`.
> - `sre` → `AdminAccess` in `prod`.
> - Engineers run `aws sso login` once a day; assumed roles are short-lived (1-12h).

## Usage

Mostly console-driven during setup. After:

```bash
# Configure SSO profile
aws configure sso --profile dev
# Goes through browser flow, picks account + role

# Then use the profile
aws s3 ls --profile dev
```

`~/.aws/config`:
```ini
[profile dev]
sso_session = mycompany
sso_account_id = 123456789012
sso_role_name = DeveloperAccess
region = ap-south-1

[sso-session mycompany]
sso_start_url = https://d-xxxx.awsapps.com/start
sso_region = ap-south-1
sso_registration_scopes = sso:account:access
```

## Pricing

- **Free.**

## Gotchas

- **Region-pinned.** Pick wisely — moving the directory later is painful.
- **External IdP setup** (Okta, Entra) needs SAML + SCIM provisioning configured carefully.
- **Sessions max 12 hours.** That's a feature.
- **Some services / APIs are slow to support Identity Center** — check edge cases.

## Related

- [IAM](./iam.md)
- [Organizations](../11-cost-management/organizations.md)
