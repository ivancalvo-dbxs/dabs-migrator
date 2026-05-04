# dabs-migrator — change log

For future agents iterating on this skill: every entry below captures **why** a rule exists, not just what changed. When you tweak conventions, add an entry here so the reasoning survives.

Entries are reverse-chronological. Each entry: date, what changed, **why** (with the failure mode that drove the change when applicable), where the rule lives now.

---

## 2026-05-03 — Pipeline library entry kind: per-extension, not blanket

**Change:** Reverted the earlier blanket "always `notebook:`" rule for `pipelines` libraries. New rule: `.py` sources use `notebook:`, `.sql` sources use `file:`.

**Why:** The previous rule (always `notebook:`) was overcorrected. SQL pipelines need `file:` entries; using `notebook:` for `.sql` produces the symmetric "expected a notebook" error. The real invariant is *match the entry kind to the file extension*.

**Where:**
- `resources/pipelines.md` — kind-by-extension table + mixed example.
- `SKILL.md` hard rule — banned *mismatch* between kind and extension, citing both error directions.

---

## 2026-05-03 — Clone-verbatim rule for migrated source files

**Change:** When migrating an existing Databricks asset, source files under `src/<name>/` must be cloned verbatim from the original notebook/script. Stub code, blueprints, sample logic, `# TODO: Replace with actual ingestion logic` placeholders, and `# Originally sourced from: <path>` headers are banned.

**Why:** During real migration, the agent was generating placeholder bodies and TODO comments instead of preserving the user's working logic. This silently breaks production behavior — the bundle deploys, but the pipeline/job no longer does what it did before. Stubs are only acceptable when the user explicitly says they're starting from scratch.

**Where:**
- `SKILL.md` workflow step 4 — split into "migrating existing" (default, clone verbatim) vs. "from scratch" (stubs).
- `SKILL.md` hard rules — explicit ban with the failure mode named.
- `resources/jobs.md`, `resources/pipelines.md`, `resources/apps.md`, `resources/dashboards.md` — replaced "Source code stubs" sections with "migrating existing → clone verbatim" + "from scratch → stubs" branches.

---

## 2026-05-03 — Pipeline `target:` → `schema:` (SDP nomenclature)

**Change:** Pipeline skeletons now use `schema:` instead of `target:`.

**Why:** `target` is the legacy DLT field. New Spark Declarative Pipelines (SDP, the rebranded Lakeflow Declarative Pipelines / DLT) use `schema`. `target` may still work but emits deprecation warnings and is being phased out.

**Where:**
- `resources/pipelines.md` — skeleton + dedicated "Why `schema:` and not `target:`" section.
- `SKILL.md` hard rule — never use `target:` on new pipelines.

---

## 2026-05-03 — Relative path fix: `../src/` → `../../src/`

**Change:** All relative `path:` / `notebook_path:` / `file_path:` / `source_code_path:` values in resource YAML skeletons now use `../../src/<name>/...` instead of `../src/<name>/...`.

**Why:** Resource files live at `resources/<type>/<name>.yml` — two levels under the bundle root. Going up only one level (`../`) resolves to `resources/`, not the bundle root, so `../src/...` resolves to `resources/src/...` and the deploy fails to find the source files. Two levels up (`../../`) is correct.

**Where:**
- `resources/jobs.md` (notebook_task path + SQL task variation).
- `resources/pipelines.md` (libraries paths + dedicated "Path convention" section).
- `resources/apps.md` (source_code_path).
- `resources/dashboards.md` (file_path).
- `SKILL.md` hard rule — banned single-`..` paths with the failure mode.

---

## 2026-04-29 — Initial scaffold

**Change:** Created the `dabs-migrator` skill from scratch.

**Why:** Hackathon goal — a single Databricks workspace skill that takes a list of resource names and produces a complete DABs project repo (bundle root, per-resource YAML, source stubs, tests, CI/CD pipelines).

**Layout established:**

```
dabs-migrator/
├── SKILL.md                      # entrypoint with workflow + hard rules
├── CHANGELOG.md                  # this file
├── resources/                    # one .md per supported DABs resource (23 total)
│   ├── alerts.md  apps.md  catalogs.md  clusters.md  dashboards.md
│   ├── database_catalogs.md  database_instances.md  experiments.md
│   ├── external_locations.md  jobs.md  models.md
│   ├── model_serving_endpoints.md  pipelines.md
│   ├── postgres_branches.md  postgres_endpoints.md  postgres_projects.md
│   ├── quality_monitors.md  registered_models.md  schemas.md
│   ├── secret_scopes.md  sql_warehouses.md  synced_database_tables.md
│   └── volumes.md
├── cicd/                         # one .md per supported CI/CD tool
│   ├── github-actions.md (default)  azure-devops.md  gitlab-ci.md
│   ├── bitbucket.md  jenkins.md  circleci.md
└── templates/                    # shared file templates
    ├── databricks.yml.tmpl       # bundle entrypoint
    ├── gitignore.tmpl
    ├── requirements.txt.tmpl
    └── README.md.tmpl
```

**Core conventions established at creation:**

- One asset per resource file. Never bundle multiple jobs into one YAML.
- Folder name in `src/` must equal the resource key in `resources/`.
- Never hardcode workspace hosts / cluster IDs / warehouse IDs / catalog names — use bundle variables and per-target overrides.
- Never commit secrets. Use the CI tool's secret store + Databricks secret scopes.
- CI/CD action contract for every generated pipeline: install CLI → `bundle validate --output json` → `bundle deploy -t <target>` (deploy only on the deploy pipelines).
- `databricks repos` commands are completely banned — bundle deploy handles workspace sync, mixing them creates dual sources of truth.
- Production deploys go through CI only, never from a dev machine.

**Source of truth references:**
- Supported resources: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#supported-resources
- Bundle jobs tutorial (CI/CD reference): https://docs.databricks.com/aws/en/dev-tools/bundles/jobs-tutorial
- Skill format spec: https://docs.databricks.com/aws/en/genie-code/skills
- Canonical example projects:
  - https://github.com/databricks-solutions/databricks-dab-examples/tree/main/sts-dabs-demo
  - https://github.com/databricks-solutions/databricks-dab-examples/tree/main/flights/flights-simple
  - https://github.com/databricks-solutions/databricks-dab-examples/tree/main/flights/flights-advanced

---

## How to add an entry

When you change a rule or skeleton, prepend a new section here with:

1. **Date** — ISO format (`YYYY-MM-DD`).
2. **One-line change summary** in the heading.
3. **Why** — the reason the rule exists. If a specific error message or failure drove it, paste the literal error so future agents can grep for it.
4. **Where** — the files touched, so the next agent can audit consistency.

Keep entries short but include enough context that the rule could be re-derived from this log alone.
