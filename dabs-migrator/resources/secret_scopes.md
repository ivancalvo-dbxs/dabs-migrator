# Resource: `secret_scopes`

Databricks secret scopes (containers for secrets).

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#secret_scope

## Skeleton

```yaml
resources:
  secret_scopes:
    {{ scope_name }}:
      name: {{ scope_name }}
      backend_type: DATABRICKS    # or AZURE_KEYVAULT
      permissions:
        - level: READ
          group_name: data-engineers
```

## Hard rules

- **Never** put secret values in YAML or source. The bundle creates the scope; populate values out-of-band via `databricks secrets put-secret` or your secret manager.
- Reference secrets at runtime as `{{secrets/<scope>/<key>}}` from job/pipeline configs.

## What to ask the user

- Databricks-backed scope or external (Azure Key Vault)?
- Which principals get READ vs MANAGE?
