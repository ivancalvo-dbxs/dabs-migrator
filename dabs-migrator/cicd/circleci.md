# CI/CD: CircleCI

`.circleci/config.yml` at repo root.

## Required project environment variables

| Variable |
|---|
| `DATABRICKS_HOST_STAGING` |
| `DATABRICKS_HOST_PROD` |
| `DATABRICKS_TOKEN` |

## `.circleci/config.yml`

```yaml
version: 2.1

executors:
  ubuntu:
    docker:
      - image: cimg/base:current

commands:
  install-cli:
    steps:
      - run:
          name: Install Databricks CLI
          command: curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sudo sh

jobs:
  validate:
    executor: ubuntu
    steps:
      - checkout
      - install-cli
      - run:
          name: Validate bundle
          environment:
            DATABRICKS_HOST: $DATABRICKS_HOST_STAGING
          command: databricks bundle validate --output json | tee bundle-validate.json
      - store_artifacts:
          path: bundle-validate.json

  deploy-staging:
    executor: ubuntu
    steps:
      - checkout
      - install-cli
      - run:
          environment:
            DATABRICKS_HOST: $DATABRICKS_HOST_STAGING
          command: |
            databricks bundle validate --output json
            databricks bundle deploy -t staging

  deploy-prod:
    executor: ubuntu
    steps:
      - checkout
      - install-cli
      - run:
          environment:
            DATABRICKS_HOST: $DATABRICKS_HOST_PROD
          command: |
            databricks bundle validate --output json
            databricks bundle deploy -t prod

workflows:
  validate-and-deploy:
    jobs:
      - validate
      - deploy-staging:
          requires: [validate]
          filters:
            branches:
              only: main
      - hold-prod:
          type: approval
          filters:
            tags:
              only: /^v.*/
            branches:
              ignore: /.*/
      - deploy-prod:
          requires: [hold-prod]
          filters:
            tags:
              only: /^v.*/
            branches:
              ignore: /.*/
```

## Notes

- `type: approval` on `hold-prod` is the manual gate.
- CircleCI requires explicit tag filters on every job in a tag-triggered workflow — copy the `filters:` block.
