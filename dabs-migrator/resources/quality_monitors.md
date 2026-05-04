# Resource: `quality_monitors`

Lakehouse Monitoring — data/ML quality monitors on UC tables.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#quality_monitor

## Skeleton

```yaml
resources:
  quality_monitors:
    {{ monitor_name }}:
      table_name: ${var.catalog}.{{ schema_name }}.{{ table_name }}
      assets_dir: /Workspace/Shared/monitors/${bundle.name}/{{ monitor_name }}
      output_schema_name: ${var.catalog}.monitoring

      schedule:
        quartz_cron_expression: "0 0 8 * * ?"
        timezone_id: UTC

      snapshot: {}
      # OR for time series:
      # time_series:
      #   granularities: ["1 day"]
      #   timestamp_col: event_ts

      notifications:
        on_failure:
          email_addresses:
            - ${var.notification_email}
```

## What to ask the user

- Monitor type (snapshot, time series, inference log)?
- Granularity and timestamp column (if time series)?
