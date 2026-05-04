# Resource: `pipelines`

Spark Declarative Pipelines (SDP) — formerly Lakeflow Declarative Pipelines / Delta Live Tables (DLT). One YAML per pipeline under `resources/pipelines/<name>.yml`.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#pipeline  
Pipelines API: https://docs.databricks.com/api/workspace/pipelines/create

## Skeleton

```yaml
resources:
  pipelines:
    {{ pipeline_name }}:
      name: {{ pipeline_name }}
      catalog: ${var.catalog}
      schema: {{ pipeline_name }}_db        # SDP: use `schema`, not the legacy `target`
      channel: CURRENT
      photon: true
      development: ${bundle.target == "dev"}
      continuous: false

      libraries:
        - notebook:
            path: ../../src/{{ pipeline_name }}/bronze.py
        - notebook:
            path: ../../src/{{ pipeline_name }}/silver.py
        - notebook:
            path: ../../src/{{ pipeline_name }}/gold.py

      clusters:
        - label: default
          autoscale:
            min_workers: 1
            max_workers: 4
            mode: ENHANCED

      configuration:
        bundle.sourcePath: ${workspace.file_path}/src/{{ pipeline_name }}

      permissions:
        - level: CAN_MANAGE
          user_name: ${workspace.current_user.userName}
```

## Library entry kind: `notebook:` vs `file:`

Pick the entry kind by file extension:

| Source extension | Entry kind |
|---|---|
| `.py` | `notebook:` |
| `.sql` | `file:` |

Mismatching the kind produces:

```
Error: expected a file for "resources.pipelines.<name>.libraries[0].file.path"
```

(or the symmetric `notebook` error). Always match the kind to the file extension — never use `file:` for a Python source, and never use `notebook:` for a `.sql` source.

Mixed example:

```yaml
libraries:
  - notebook:
      path: ../../src/{{ pipeline_name }}/bronze.py
  - file:
      path: ../../src/{{ pipeline_name }}/silver.sql
```

## Why `schema:` and not `target:`

`target` is the legacy DLT field name. New SDP pipelines should use `schema`. Using `target` may still work but emits deprecation warnings and is being phased out.

## Path convention

All `path:` values are relative to the bundle root **resolved via the YAML file's location**. Since pipeline YAMLs live at `resources/pipelines/<name>.yml`, paths to `src/` need **two** `..` segments: `../../src/<name>/...`.

## Source code

**If migrating an existing pipeline:** clone every notebook/script the source pipeline loaded as a library, verbatim, into `src/{{ pipeline_name }}/`. Preserve filenames, decorators, SQL bodies, and comments exactly. Do not replace any block with `# TODO: Replace with actual ingestion logic` or add `# Originally sourced from:` headers. Update each `libraries[].notebook.path` entry above to match the cloned filenames.

**Stubs (only when starting from scratch)** — medallion convention:

- `bronze.py` — raw ingestion (`@dlt.table` reading from sources).
- `silver.py` — cleansed/typed tables.
- `gold.py` — aggregated business tables.

Each should `import dlt` and define `@dlt.table`-decorated functions.

## Common variations

- **Continuous pipeline** — set `continuous: true`; omit any external scheduler.
- **Serverless** — drop the `clusters:` block; set `serverless: true`.
- **SQL pipeline** — replace `.py` libraries with `.sql` files and switch the entry kind to `file:` (one `file:` entry per `.sql` source).

## What to ask the user

- Catalog and target schema?
- Triggered or continuous?
- Serverless or classic compute?
