# Resource: `database_catalogs`

Registers a Lakebase Postgres database as a Unity Catalog catalog so its tables are queryable from Databricks.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#database_catalog

## Skeleton

```yaml
resources:
  database_catalogs:
    {{ catalog_name }}:
      name: {{ catalog_name }}
      database_instance_name: {{ database_instance_name }}
      database_name: {{ database_name }}
      create_database_if_not_exists: true
```

## What to ask the user

- Which Lakebase database instance and database name?
- Should the bundle create the DB if missing?
