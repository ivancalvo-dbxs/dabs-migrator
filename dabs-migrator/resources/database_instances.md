# Resource: `database_instances`

Lakebase Postgres database instances (managed Postgres).

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#database_instance

## Skeleton

```yaml
resources:
  database_instances:
    {{ instance_name }}:
      name: {{ instance_name }}
      capacity: CU_1               # CU_1, CU_2, CU_4, ...
      stopped: false
      retention_window_in_days: 7
```

## What to ask the user

- Capacity (compute units)?
- Retention window for PITR?
