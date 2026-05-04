# Resource: `dashboards`

AI/BI Dashboards (Lakeview).

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#dashboard

## Skeleton

```yaml
resources:
  dashboards:
    {{ dashboard_name }}:
      display_name: {{ dashboard_name }}
      file_path: ../../src/{{ dashboard_name }}/dashboard.lvdash.json
      warehouse_id: ${var.warehouse_id}
      parent_path: /Workspace/Shared/dashboards

      permissions:
        - level: CAN_MANAGE
          user_name: ${workspace.current_user.userName}
```

## Source

**If migrating an existing dashboard:** export the original Lakeview dashboard JSON from the workspace and place it verbatim at `src/{{ dashboard_name }}/dashboard.lvdash.json`. Do not edit, prettify, or simplify the exported JSON — bundle deploy round-trips it as-is.

**Stub (only when starting from scratch):** create an empty `src/{{ dashboard_name }}/dashboard.lvdash.json`, then author in the UI and export back into the file.

## What to ask the user

- Existing dashboard to export, or build new?
- Which warehouse runs the queries?
