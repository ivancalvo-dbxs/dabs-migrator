# Resource: `sql_warehouses`

SQL warehouses (compute endpoints for SQL).

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#sql_warehouse

## Skeleton

```yaml
resources:
  sql_warehouses:
    {{ warehouse_name }}:
      name: {{ warehouse_name }}
      cluster_size: Small
      auto_stop_mins: 10
      min_num_clusters: 1
      max_num_clusters: 3
      enable_serverless_compute: true
      warehouse_type: PRO
      channel:
        name: CHANNEL_NAME_CURRENT

      permissions:
        - level: CAN_USE
          group_name: data-analysts
```

## What to ask the user

- Serverless, Pro, or Classic?
- T-shirt size (XS, S, M, L, XL, ...) and autoscale bounds?
- Auto-stop minutes?
