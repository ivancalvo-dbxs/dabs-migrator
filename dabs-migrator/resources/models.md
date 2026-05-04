# Resource: `models`

**Legacy** workspace model registry. Prefer `registered_models` (Unity Catalog) for new work — only use `models` if the workspace is not Unity Catalog enabled.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#model

## Skeleton

```yaml
resources:
  models:
    {{ model_name }}:
      name: {{ model_name }}
      description: "Legacy registry model managed by ${bundle.name}"
      tags:
        - key: project
          value: ${bundle.name}

      permissions:
        - level: CAN_MANAGE
          group_name: ml-engineers
```

## What to ask the user

- Is this UC-enabled? If yes, use `registered_models` instead.
