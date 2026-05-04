# Resource: `synced_database_tables`

Tables synced from a Databricks UC table to a Lakebase Postgres database.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#synced_database_table

## Skeleton

```yaml
resources:
  synced_database_tables:
    {{ sync_name }}:
      name: {{ sync_name }}
      database_instance_name: {{ database_instance_name }}
      logical_database_name: {{ database_name }}
      spec:
        source_table_full_name: ${var.catalog}.{{ schema_name }}.{{ source_table }}
        primary_key_columns: [id]
        scheduling_policy: CONTINUOUS    # or TRIGGERED
```

## What to ask the user

- Source UC table?
- Target Lakebase database instance + logical database?
- Primary key column(s)?
- Continuous or triggered sync?
