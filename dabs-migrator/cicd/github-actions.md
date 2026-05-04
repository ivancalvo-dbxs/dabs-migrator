# CI/CD: GitHub Actions

Default CI/CD tool. Generates three workflows under `.github/workflows/`.

Reference: https://docs.databricks.com/aws/en/dev-tools/bundles/jobs-tutorial

## Required repo secrets

| Secret | Where used | Notes |
|---|---|---|
| `DATABRICKS_HOST_STAGING` | staging deploy | workspace URL |
| `DATABRICKS_HOST_PROD` | prod deploy | workspace URL |
| `DATABRICKS_TOKEN` *or* OIDC | both | prefer OIDC federation to an Azure AD / AWS IAM service principal |

## `.github/workflows/pr_validate.yml`

```yaml
name: Validate bundle
on:
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Databricks CLI
        uses: databricks/setup-cli@main

      - name: Validate bundle
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST_STAGING }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
        run: databricks bundle validate --output json | tee bundle-validate.json

      - uses: actions/upload-artifact@v4
        with:
          name: bundle-validate
          path: bundle-validate.json
```

## `.github/workflows/deploy_to_staging.yml`

```yaml
name: Deploy to staging
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Install Databricks CLI
        uses: databricks/setup-cli@main

      - name: Validate bundle
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST_STAGING }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
        run: databricks bundle validate --output json

      - name: Deploy bundle
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST_STAGING }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
        run: databricks bundle deploy -t staging
```

## `.github/workflows/deploy_to_prod.yml`

```yaml
name: Deploy to prod
on:
  push:
    tags: ["v*"]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: prod   # configure required reviewers in repo settings
    steps:
      - uses: actions/checkout@v4

      - name: Install Databricks CLI
        uses: databricks/setup-cli@main

      - name: Validate bundle
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST_PROD }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
        run: databricks bundle validate --output json

      - name: Deploy bundle
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST_PROD }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
        run: databricks bundle deploy -t prod
```

## Notes

- Use a GitHub `environment:` for `prod` so you can require manual approval / restrict to specific reviewers.
- Never run `databricks repos` in any of these workflows — bundle deploy handles workspace sync.
- Token-based auth shown above; for OIDC swap `DATABRICKS_TOKEN` for `azure/login@v2` (or AWS equivalent) and let the CLI pick up federated credentials.
