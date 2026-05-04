# Resource: `clusters`

All-purpose compute clusters. Generally **prefer job clusters** (defined inline in `jobs`) — only declare standalone clusters when multiple humans/jobs share them.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#cluster

## Skeleton

```yaml
resources:
  clusters:
    {{ cluster_name }}:
      cluster_name: {{ cluster_name }}
      spark_version: 15.4.x-scala2.12
      node_type_id: i3.xlarge
      num_workers: 2
      autotermination_minutes: 30
      data_security_mode: USER_ISOLATION

      permissions:
        - level: CAN_RESTART
          group_name: data-engineers
```

## What to ask the user

- Single-user or shared (USER_ISOLATION)?
- Fixed workers or autoscale?
- Init scripts or custom libraries?
