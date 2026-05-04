# Resource: `alerts`

SQL Alerts (Alerts V2) — monitor query results and notify on threshold breaches.

Docs: https://docs.databricks.com/api/workspace/alertsv2/createalert

## Skeleton

```yaml
resources:
  alerts:
    {{ alert_name }}:
      display_name: {{ alert_name }}
      query_text: |
        SELECT count(*) AS bad_rows
        FROM ${var.catalog}.monitoring.errors
        WHERE event_date = current_date() - 1
      warehouse_id: ${var.warehouse_id}
      parent_path: /Workspace/Shared/alerts

      schedule:
        quartz_cron_schedule: "0 0 9 * * ?"
        timezone_id: UTC
        pause_status: UNPAUSED

      evaluation:
        comparison_operator: GREATER_THAN
        empty_result_state: OK
        source:
          name: bad_rows
          aggregation: SUM
        threshold:
          value:
            double_value: 0
        notification:
          notify_on_ok: true
          retrigger_seconds: 3600
          subscriptions:
            - user_email: ${var.notification_email}

      permissions:
        - level: CAN_MANAGE
          user_name: ${workspace.current_user.userName}
```

## What to ask the user

- Query and threshold (operator + value)?
- Schedule cron + timezone?
- Notification recipients (emails or destination IDs)?
