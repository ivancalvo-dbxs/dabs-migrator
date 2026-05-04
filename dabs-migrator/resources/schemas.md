# Resource: `schemas`

Unity Catalog schemas (databases) within a catalog.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#schema

## Skeleton

```yaml
resources:
  schemas:
    {{ schema_name }}:
      catalog_name: ${var.catalog}
      name: {{ schema_name }}
      comment: "Schema managed by ${bundle.name}"
      grants:
        - principal: data-engineers
          privileges: [USE_SCHEMA, CREATE_TABLE, MODIFY]
```

## What to ask the user

- Parent catalog?
- Initial grants?
