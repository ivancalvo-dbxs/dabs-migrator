# Resource: `postgres_endpoints`

Lakebase Autoscaling compute endpoint — connection point for a branch.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#postgres_endpoint

## Skeleton

```yaml
resources:
  postgres_endpoints:
    {{ endpoint_name }}:
      name: {{ endpoint_name }}
      project_name: {{ project_name }}
      branch_name: {{ branch_name }}
      type: read_write             # or read_only
      autoscaling_limit_min_cu: 0.25
      autoscaling_limit_max_cu: 4
      suspend_timeout_seconds: 300
```

## What to ask the user

- Read-write or read-only?
- Min/max CU and suspend timeout?
