---
name: databricks-project-structure
description: Use when adding, modifying, or locating files in a Databricks Asset Bundle (DABS) project — defines where jobs, pipelines, notebooks, tests, CI/CD pipelines (GitHub Actions, Azure DevOps, GitLab CI, Jenkins, etc.), and bundle config belong. Trigger when the user asks "where does X go?", creates a new job/pipeline/notebook, edits databricks.yml, or sets up a CI/CD pipeline for staging/prod deploys.
---

# Databricks Asset Bundle (DABS) Project Structure

Standard layout for a Databricks project using Asset Bundles. Every file has one canonical home — when in doubt, follow the conventions below before inventing a new location.

## Layout

```
my_project/
├── .github/                           # CI/CD config-as-code (example: GitHub Actions)
│   └── workflows/                     # could equally be .azure-pipelines/, .gitlab-ci.yml,
│       ├── deploy_to_staging.yml      # Jenkinsfile, .circleci/, etc. — see "CI/CD" section
│       ├── deploy_to_prod.yml
│       └── ...
├── resources/
│   ├── jobs/
│   │   └── my_job_1.yml               # one YAML per job (DABS resource)
│   └── pipelines/
│       └── my_pipeline_1.yml          # one YAML per Lakeflow Declarative pipeline
├── src/
│   ├── my_job_1/                      # source files for my_job_1
│   │   ├── notebook_1.py
│   │   ├── notebook_2.py
│   │   └── create_metric_view.sql
│   ├── my_pipeline_2/                 # medallion-style pipeline source
│   │   ├── bronze.py
│   │   ├── silver.py
│   │   └── gold.py
│   └── ...
├── tests/
│   └── my_unit_testing_1.py           # pytest-style unit tests
├── databricks.yml                     # bundle entrypoint: name, targets, includes
└── requirements.txt                   # Python deps for local dev + tests
```

## Folder responsibilities

### `databricks.yml` (root)
Bundle entrypoint. Declares `bundle.name`, `targets` (e.g. `dev`, `staging`, `prod`), workspace hosts, and `include:` globs that pull in `resources/**/*.yml`. Do **not** define jobs or pipelines inline here — keep the root file thin and delegate to `resources/`.

### `resources/jobs/`
One YAML file per Databricks Job. The file declares the `resources.jobs.<key>` block: tasks, schedules, clusters, parameters, permissions. Tasks reference notebooks/scripts via paths under `src/`. Naming: `<job_name>.yml` matching the job key.

### `resources/pipelines/`
One YAML file per Lakeflow Declarative pipeline (formerly DLT). Declares `resources.pipelines.<key>`: target schema, libraries (pointing at `src/<pipeline>/*.py`), continuous/triggered, channel, photon, etc.

### `src/<asset_name>/`
Source code for a job or pipeline, grouped by the asset that owns it. Folder name **must match** the job/pipeline key in `resources/`. Mixed languages (`.py`, `.sql`, `.ipynb`) are fine — keep them co-located with the asset that runs them. Shared utilities go in a separate package (e.g. `src/common/`), not duplicated across asset folders.

### `tests/`
Unit tests (pytest) for code in `src/`. Tests should not require a live workspace — mock `dbutils`, Spark sessions, and external services. Integration tests that hit a real workspace belong in CI and should run against the `staging` target only.

### `.github/workflows/` — CI/CD (config-as-code)
This example uses **GitHub Actions**, but the structure is tool-agnostic. Any CI/CD system that supports configuration-as-code works equally well — pick the one your org already uses:

| Tool | Conventional location |
|---|---|
| GitHub Actions | `.github/workflows/*.yml` |
| Azure DevOps Pipelines | `azure-pipelines.yml` or `.azure-pipelines/*.yml` |
| GitLab CI | `.gitlab-ci.yml` |
| Bitbucket Pipelines | `bitbucket-pipelines.yml` |
| Jenkins | `Jenkinsfile` (declarative) |
| CircleCI | `.circleci/config.yml` |
| Databricks Workflows | bundle resource itself (self-deploying) |

**Hard requirement:** the pipeline definition must live in the repo and be version-controlled. ClickOps pipelines configured in a UI are out — they can't be reviewed, rolled back, or replicated across environments.

**Pipelines you typically need, regardless of tool:**
- **Staging deploy** — on merge to `main`, runs `databricks bundle validate` then `databricks bundle deploy -t staging`.
- **Prod deploy** — on tag/release, deploys to `prod` with manual approval / protected environment gates.
- **PR validation** — `databricks bundle validate`, unit tests (`pytest`), lint (ruff/black), bundle schema checks.

**Auth:** prefer OIDC / federated identity (GitHub OIDC → Azure AD service principal, Azure DevOps workload identity federation, etc.) over long-lived PATs. Whatever tool you use, the secret/credential it uses to call Databricks should be short-lived and scoped to the target workspace.

### `requirements.txt`
Python dependencies for **local development and tests only**. Runtime dependencies for jobs/pipelines should be declared in the bundle's `environments` / cluster `libraries` blocks, not assumed from this file.

## Conventions and rules

1. **One asset per resource file.** Don't bundle multiple jobs into a single YAML. Easier to review, diff, and own.
2. **Folder name = resource key.** `src/my_job_1/` ↔ `resources/jobs/my_job_1.yml` with key `my_job_1`. Renames must update both sides.
3. **Paths in resource YAML are relative to the bundle root.** Use `./src/my_job_1/notebook_1.py`, not absolute workspace paths.
4. **Never hardcode workspace hosts, cluster IDs, or warehouse IDs in resource YAML.** Use bundle variables (`${var.warehouse_id}`) and target overrides.
5. **Secrets never in YAML or source.** Reference Databricks secret scopes (`{{secrets/scope/key}}`) from job/pipeline configs.
6. **CI deploys, humans don't.** Production deploys go through the prod pipeline in your CI/CD tool of choice, never `databricks bundle deploy -t prod` from a laptop.

## Where does X go? (quick reference)

| You're adding... | It goes in... |
|---|---|
| A new scheduled job | `resources/jobs/<name>.yml` + `src/<name>/` |
| A new DLT/Lakeflow pipeline | `resources/pipelines/<name>.yml` + `src/<name>/` |
| A notebook used by an existing job | `src/<existing_job>/` |
| A SQL view/UDF for an existing job | `src/<existing_job>/*.sql` |
| Shared Python helpers | `src/common/` (new) — import from multiple assets |
| A unit test | `tests/test_<thing>.py` |
| A new deployment environment | new `targets:` entry in `databricks.yml` + matching pipeline in your CI/CD tool's config dir |
| A new CI check | new pipeline file in your CI/CD tool's config dir (e.g. `.github/workflows/<check>.yml`, `.azure-pipelines/<check>.yml`) |
| A Python library dependency for local dev | `requirements.txt` |
| A Python library dependency for the cluster | `environments` / `libraries` block in the resource YAML |
