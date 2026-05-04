# Resource: `postgres_branches`

Lakebase Autoscaling branch — copy-on-write Postgres branch within a project.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#postgres_branch

## Skeleton

```yaml
resources:
  postgres_branches:
    {{ branch_name }}:
      name: {{ branch_name }}
      project_name: {{ project_name }}
      parent_branch_name: main
      protected: false
```

## What to ask the user

- Parent project + parent branch?
- Protected (no deletion) for prod branches?
