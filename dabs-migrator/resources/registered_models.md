# Resource: `registered_models`

Unity Catalog registered models (preferred over legacy `models`).

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#registered_model

## Skeleton

```yaml
resources:
  registered_models:
    {{ model_name }}:
      catalog_name: ${var.catalog}
      schema_name: ml_models
      name: {{ model_name }}
      comment: "Registered model managed by ${bundle.name}"
      grants:
        - principal: ml-engineers
          privileges: [EXECUTE, APPLY_TAG]
```

## What to ask the user

- Catalog and schema?
- Who needs EXECUTE (inference) vs MANAGE?
