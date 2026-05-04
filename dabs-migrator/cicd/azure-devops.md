# CI/CD: Azure DevOps Pipelines

Generates three pipelines under `.azure-pipelines/`.

## Required variables (variable group `databricks-bundle`)

| Variable | Notes |
|---|---|
| `DATABRICKS_HOST_STAGING` | workspace URL |
| `DATABRICKS_HOST_PROD` | workspace URL |
| `DATABRICKS_TOKEN` | secret variable, or use workload identity federation |

## `.azure-pipelines/pr-validate.yml`

```yaml
trigger: none
pr:
  branches:
    include: [main]

pool:
  vmImage: ubuntu-latest

variables:
  - group: databricks-bundle

steps:
  - script: curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
    displayName: Install Databricks CLI

  - script: databricks bundle validate --output json | tee bundle-validate.json
    displayName: Validate bundle
    env:
      DATABRICKS_HOST: $(DATABRICKS_HOST_STAGING)
      DATABRICKS_TOKEN: $(DATABRICKS_TOKEN)

  - publish: bundle-validate.json
    artifact: bundle-validate
```

## `.azure-pipelines/deploy-staging.yml`

```yaml
trigger:
  branches:
    include: [main]
pr: none

pool:
  vmImage: ubuntu-latest

variables:
  - group: databricks-bundle

stages:
  - stage: Deploy
    jobs:
      - deployment: staging
        environment: staging
        strategy:
          runOnce:
            deploy:
              steps:
                - checkout: self
                - script: curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
                  displayName: Install Databricks CLI
                - script: databricks bundle validate --output json
                  displayName: Validate bundle
                  env:
                    DATABRICKS_HOST: $(DATABRICKS_HOST_STAGING)
                    DATABRICKS_TOKEN: $(DATABRICKS_TOKEN)
                - script: databricks bundle deploy -t staging
                  displayName: Deploy bundle
                  env:
                    DATABRICKS_HOST: $(DATABRICKS_HOST_STAGING)
                    DATABRICKS_TOKEN: $(DATABRICKS_TOKEN)
```

## `.azure-pipelines/deploy-prod.yml`

```yaml
trigger:
  tags:
    include: ["v*"]
pr: none

pool:
  vmImage: ubuntu-latest

variables:
  - group: databricks-bundle

stages:
  - stage: Deploy
    jobs:
      - deployment: prod
        environment: prod   # add approvals/checks in the environment settings
        strategy:
          runOnce:
            deploy:
              steps:
                - checkout: self
                - script: curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
                  displayName: Install Databricks CLI
                - script: databricks bundle validate --output json
                  displayName: Validate bundle
                  env:
                    DATABRICKS_HOST: $(DATABRICKS_HOST_PROD)
                    DATABRICKS_TOKEN: $(DATABRICKS_TOKEN)
                - script: databricks bundle deploy -t prod
                  displayName: Deploy bundle
                  env:
                    DATABRICKS_HOST: $(DATABRICKS_HOST_PROD)
                    DATABRICKS_TOKEN: $(DATABRICKS_TOKEN)
```

## Notes

- Configure approvals on the `prod` Environment in Azure DevOps Project Settings → Environments.
- Prefer **workload identity federation** (Azure DevOps service connection → Azure AD app) over storing `DATABRICKS_TOKEN`.
