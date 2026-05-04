# Resource: `apps`

Databricks Apps (Streamlit, Dash, Flask, Gradio, etc.).

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#app

## Skeleton

```yaml
resources:
  apps:
    {{ app_name }}:
      name: {{ app_name }}
      description: "{{ app_name }} application"
      source_code_path: ../../src/{{ app_name }}

      resources:
        - name: warehouse
          sql_warehouse:
            id: ${var.warehouse_id}
            permission: CAN_USE

      permissions:
        - level: CAN_MANAGE
          user_name: ${workspace.current_user.userName}
```

## Source code

**If migrating an existing app:** clone the entire app source tree (entrypoint, modules, `app.yaml`, `requirements.txt`, static assets) verbatim into `src/{{ app_name }}/`. Preserve directory structure and filenames. Do not replace any module with stub code or `# TODO` placeholders.

**Stubs (only when starting from scratch):**

- `app.yaml` — app config (command, env vars).
- `app.py` — entrypoint.
- `requirements.txt` — app-specific Python deps.

## What to ask the user

- Framework (Streamlit, Dash, Flask, Gradio)?
- Which workspace resources does the app need (warehouses, secrets, serving endpoints)?
