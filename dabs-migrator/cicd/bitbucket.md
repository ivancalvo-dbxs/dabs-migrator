# CI/CD: Bitbucket Pipelines

Single `bitbucket-pipelines.yml` at repo root.

## Required repository variables

| Variable | Secured |
|---|---|
| `DATABRICKS_HOST_STAGING` | yes |
| `DATABRICKS_HOST_PROD` | yes |
| `DATABRICKS_TOKEN` | yes |

## `bitbucket-pipelines.yml`

```yaml
image: ubuntu:24.04

definitions:
  steps:
    - step: &install-cli
        name: Install Databricks CLI
        script:
          - apt-get update && apt-get install -y curl ca-certificates
          - curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh

    - step: &validate
        name: Validate bundle
        script:
          - apt-get update && apt-get install -y curl ca-certificates
          - curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
          - export DATABRICKS_HOST=$DATABRICKS_HOST_STAGING
          - databricks bundle validate --output json | tee bundle-validate.json
        artifacts:
          - bundle-validate.json

pipelines:
  pull-requests:
    "**":
      - step: *validate

  branches:
    main:
      - step: *validate
      - step:
          name: Deploy to staging
          deployment: staging
          script:
            - apt-get update && apt-get install -y curl ca-certificates
            - curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
            - export DATABRICKS_HOST=$DATABRICKS_HOST_STAGING
            - databricks bundle validate --output json
            - databricks bundle deploy -t staging

  tags:
    "v*":
      - step:
          name: Deploy to prod
          deployment: production
          trigger: manual
          script:
            - apt-get update && apt-get install -y curl ca-certificates
            - curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
            - export DATABRICKS_HOST=$DATABRICKS_HOST_PROD
            - databricks bundle validate --output json
            - databricks bundle deploy -t prod
```

## Notes

- `trigger: manual` on the prod step requires an explicit click in the Bitbucket UI.
- Use **Deployment variables** scoped to `staging` / `production` for per-env hosts.
