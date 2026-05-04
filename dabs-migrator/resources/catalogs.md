# Resource: `catalogs`

Unity Catalog catalogs.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#catalog

## Skeleton

```yaml
resources:
  catalogs:
    {{ catalog_name }}:
      name: {{ catalog_name }}
      comment: "Catalog managed by ${bundle.name}"
      properties:
        owner: ${workspace.current_user.userName}
      grants:
        - principal: account users
          privileges: [USE_CATALOG]
```

## What to ask the user

- Managed location (storage root) or default?
- Initial grants?
