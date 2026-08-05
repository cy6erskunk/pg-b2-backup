# pg-b2-backup

A composite GitHub Action that dumps a Postgres database with `pg_dump`, gzips it,
and uploads it to a [Backblaze B2](https://www.backblaze.com/cloud-storage) bucket
via B2's S3-compatible API. Old backups are pruned by a configurable retention window.

Works with any Postgres connection string, including [Neon](https://neon.com).

## Usage

Add a workflow like this to any repository that needs backups
(`.github/workflows/db-backup.yml`):

```yaml
name: db-backup

on:
  schedule:
    - cron: '0 5 * * *' # daily at 05:00 UTC
  workflow_dispatch:

permissions: {}

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - uses: OWNER/pg-b2-backup@v1
        with:
          database-url: ${{ secrets.DATABASE_URL }}
          b2-application-key-id: ${{ secrets.B2_APPLICATION_KEY_ID }}
          b2-application-key: ${{ secrets.B2_APPLICATION_KEY }}
          b2-bucket-name: ${{ secrets.B2_BUCKET_NAME }}
          b2-region: eu-central-003
          pg-version: '16'
```

Replace `OWNER` with the account/org that hosts this action.

## Inputs

| Input                   | Required | Default          | Description |
| ----------------------- | -------- | ---------------- | ----------- |
| `database-url`          | yes      | —                | Postgres connection string to back up. |
| `b2-application-key-id` | yes      | —                | B2 application key ID (S3 access key id). |
| `b2-application-key`    | yes      | —                | B2 application key (S3 secret). |
| `b2-bucket-name`        | yes      | —                | Target B2 bucket. |
| `b2-region`             | yes      | —                | Middle segment of the bucket's S3 endpoint, e.g. `eu-central-003`. |
| `folder-name`           | no       | calling repo name | Prefix (folder) inside the bucket. |
| `retention-days`        | no       | `7`              | Delete `.gz` backups older than this many days. |
| `pg-version`            | no       | `17`             | PostgreSQL major version to install; must be ≥ the server version. |

## Required secrets

Set these in each consuming repository (**Settings → Secrets and variables → Actions**):

- `DATABASE_URL`
- `B2_APPLICATION_KEY_ID`
- `B2_APPLICATION_KEY`
- `B2_BUCKET_NAME`

## Notes

- `pg-version` must be **≥** your Postgres server's major version or `pg_dump` refuses to run.
- Backups are stored as `s3://<bucket>/<folder-name>/<Month-Day-Year@HH:MM:SS>.sql.gz`.
- Restore with: `gunzip -c <file>.sql.gz | psql "<connection-string>" -v ON_ERROR_STOP=1`.
