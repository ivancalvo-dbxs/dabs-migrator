# CI/CD: GitLab CI

Single `.gitlab-ci.yml` at repo root.

## Required CI/CD variables

| Variable | Scope | Masked |
|---|---|---|
| `DATABRICKS_HOST_STAGING` | staging env | yes |
| `DATABRICKS_HOST_PROD` | prod env | yes |
| `DATABRICKS_TOKEN` | both, protected | yes |

## `.gitlab-ci.yml`

```yaml
stages: [validate, deploy]

default:
  image: ubuntu:24.04
  before_script:
    - apt-get update && apt-get install -y curl ca-certificates
    - curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh

validate:
  stage: validate
  script:
    - databricks bundle validate --output json | tee bundle-validate.json
  artifacts:
    paths: [bundle-validate.json]
  variables:
    DATABRICKS_HOST: $DATABRICKS_HOST_STAGING
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

deploy_staging:
  stage: deploy
  environment:
    name: staging
  script:
    - databricks bundle validate --output json
    - databricks bundle deploy -t staging
  variables:
    DATABRICKS_HOST: $DATABRICKS_HOST_STAGING
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

deploy_prod:
  stage: deploy
  environment:
    name: prod
    action: prepare
  script:
    - databricks bundle validate --output json
    - databricks bundle deploy -t prod
  variables:
    DATABRICKS_HOST: $DATABRICKS_HOST_PROD
  rules:
    - if: $CI_COMMIT_TAG =~ /^v/
      when: manual
```

## Notes

- `when: manual` on `deploy_prod` enforces a click-to-approve gate.
- Mark `DATABRICKS_TOKEN` as **Protected** so it's only injected on protected branches/tags.
