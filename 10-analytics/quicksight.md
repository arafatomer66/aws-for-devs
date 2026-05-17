# QuickSight

**TL;DR** — Managed BI dashboards. Connect to Redshift / Athena / RDS / S3 / Salesforce, build dashboards, share with viewers. Pay per user or per session.

## What it is

A cloud BI tool similar to Tableau / Power BI / Looker. Browser-based; build dashboards and reports. SPICE (Super-fast, Parallel, In-memory Calculation Engine) caches data for fast viz.

## Key concepts

- **Data source** — connection to RDS/Athena/S3/Redshift/SaaS.
- **Dataset** — query on top of a data source, loaded into SPICE or live.
- **Analysis** — your working dashboard.
- **Dashboard** — published view.
- **SPICE** — in-memory cache, $0.38/GB-mo.
- **Users:**
  - **Author** — builds.
  - **Reader** — views; pay per session (Reader Session Capacity).
  - **Admin Pro / Author Pro** — newer tiers with extra Q (Gen-AI) features.
- **Q** — natural-language Q&A on top of your data ("show sales by region this quarter").
- **Embedded analytics** — embed dashboards into your app.

## Real-world example

> Daily ops dashboard for ShareDeal:
> - Data: Athena on S3 (orders, clicks, signups) + RDS (users).
> - SPICE refreshed every hour.
> - Dashboards: GMV, orders/day, top categories, conversion funnel.
> - Embedded into the internal admin portal.

## Pricing

- **Authors:** ~$24/mo.
- **Readers:** $0.30 per session (max $5/user/mo).
- **SPICE:** $0.38/GB-mo storage.
- **Q + Pro tiers:** higher.

For embedded use, anonymous-session pricing applies.

## QuickSight vs alternatives

- **QuickSight** — AWS-native, cheap for small teams, embedded is good.
- **Tableau / Power BI** — more polished, more expensive.
- **Metabase / Superset (OSS)** — free if you self-host.

## Gotchas

- **SPICE has size limits** — older limit 250 GB per dataset.
- **Live queries** can be slow on big underlying tables — use SPICE.
- **Embedded auth via IAM Identity** can be fiddly to set up.
- **Pixel-perfect reports** are limited compared to Tableau.

## Related

- [Athena](./athena.md)
- [Redshift](../03-database/redshift.md)
- [Glue](./glue.md)
