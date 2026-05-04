# Resource: `postgres_projects`

Lakebase Autoscaling project — top-level container.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#postgres_project

## Skeleton

```yaml
resources:
  postgres_projects:
    {{ project_name }}:
      name: {{ project_name }}
      pg_version: "16"
      region: us-east-1
```

## What to ask the user

- Postgres major version?
- Region?
