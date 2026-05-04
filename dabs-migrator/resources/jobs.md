# Resource: `jobs`

Scheduled or triggered Databricks Jobs (Workflows). One YAML per job under `resources/jobs/<name>.yml`.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#job  
Job API: https://docs.databricks.com/api/workspace/jobs/create

## Skeleton

```yaml
resources:
  jobs:
    {{ job_name }}:
      name: {{ job_name }}
      max_concurrent_runs: 1
      timeout_seconds: 3600

      email_notifications:
        on_failure:
          - ${var.notification_email}

      tags:
        owner: ${workspace.current_user.userName}
        project: ${bundle.name}

      schedule:
        quartz_cron_expression: "0 0 7 * * ?"
        timezone_id: UTC
        pause_status: UNPAUSED

      job_clusters:
        - job_cluster_key: default_cluster
          new_cluster:
            spark_version: 15.4.x-scala2.12
            node_type_id: i3.xlarge
            num_workers: 2
            data_security_mode: SINGLE_USER

      tasks:
        - task_key: main
          job_cluster_key: default_cluster
          notebook_task:
            notebook_path: ../../src/{{ job_name }}/notebook.py

      permissions:
        - level: CAN_MANAGE
          user_name: ${workspace.current_user.userName}
```

## Source code

**If migrating an existing job:** clone the original notebook(s) and any referenced scripts verbatim into `src/{{ job_name }}/`. Keep filenames, comments, and logic intact. Do not insert `# TODO` placeholders or `# Originally sourced from:` headers — the user's working code must survive the migration unchanged. Update the `notebook_path` / `sql_task.file.path` entries above to match the cloned filenames.

**Stubs (only when starting from scratch):**

- `notebook.py` — entrypoint notebook referenced by the task above.
- Add `.sql` / `.py` files for additional tasks; reference each via its own task block.

## Common variations

- **Multi-task** — add more entries to `tasks:`, use `depends_on:` for ordering.
- **SQL task** — replace `notebook_task` with `sql_task: { warehouse_id: ${var.warehouse_id}, file: { path: ../../src/{{ job_name }}/query.sql } }`.
- **Python wheel task** — `python_wheel_task: { package_name: ..., entry_point: ... }` with the wheel in `environments`.
- **Triggered (no schedule)** — omit the `schedule:` block; trigger via API or another job.

## What to ask the user

- Schedule (cron) or triggered?
- Single notebook or multi-task with dependencies?
- Cluster type — job cluster (default, recommended) or existing all-purpose?
