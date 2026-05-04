# Resource: `volumes`

Unity Catalog volumes (file storage).

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#volume

## Skeleton

```yaml
resources:
  volumes:
    {{ volume_name }}:
      catalog_name: ${var.catalog}
      schema_name: {{ schema_name }}
      name: {{ volume_name }}
      volume_type: MANAGED       # or EXTERNAL
      comment: "Volume managed by ${bundle.name}"
      # storage_location only for EXTERNAL volumes
      grants:
        - principal: data-engineers
          privileges: [READ_VOLUME, WRITE_VOLUME]
```

## What to ask the user

- MANAGED (Databricks-owned storage) or EXTERNAL (BYO cloud path)?
- If EXTERNAL: which `external_location` is the parent?
