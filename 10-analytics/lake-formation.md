# Lake Formation

**TL;DR** — Builds + governs **data lakes** on S3. Fine-grained access control (column / row / cell) over Glue catalog tables, used by Athena/Redshift/EMR.

## What it is

A governance layer on top of S3 + Glue Data Catalog. Lake Formation lets you grant permissions like "user X can read columns A,B,C of table Y, but not column D, and only rows where region='ap-south-1'."

Without Lake Formation, you'd manage all this via S3 bucket policies + IAM — painful and coarse.

## Key concepts

- **Lake Formation permissions** — fine-grained grants (database / table / column / row / cell).
- **LF-tags** — tag tables/columns; grant access by tag.
- **Resource links** — share databases/tables across accounts.
- **Data Cells Filter** — row-level + column-level filters.
- **Data Lake admin** — designated principals who can manage permissions.
- **Underlying catalog** — Glue Data Catalog.

## Real-world example

> Multi-team analytics:
> - Team A can read all of `customers` table.
> - Team B can read only non-PII columns.
> - Team C can read rows only from their assigned region.
>
> All enforced through Lake Formation grants; no per-team S3 buckets.

## Usage

```bash
# Mark a Glue database as managed by Lake Formation
aws lakeformation register-resource --resource-arn arn:aws:s3:::my-lake

# Grant column-level select to a role
aws lakeformation grant-permissions \
  --principal DataLakePrincipalIdentifier=arn:aws:iam::..:role/Analyst \
  --resource '{"TableWithColumns":{"DatabaseName":"sales","Name":"customers","ColumnNames":["id","name","city"]}}' \
  --permissions SELECT
```

## Pricing

- **Lake Formation itself: free.**
- Pay for S3, Glue, Athena, etc.

## Gotchas

- **IAM + Lake Formation must both allow** the access (intersection).
- **Setup is significant** — register S3 locations, set data lake admins, opt in to Lake Formation permissions model per database.
- **Doesn't replace IAM** — works alongside.
- **Cross-account sharing** is powerful but needs the right setup (Resource Links).

## Related

- [Glue](./glue.md)
- [Athena](./athena.md)
- [Redshift](../03-database/redshift.md)
- [S3](../02-storage/s3.md)
