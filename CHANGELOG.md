# dabs-migrator — change log

For future agents iterating on this skill: every entry below captures **why** a rule exists, not just what changed. When you tweak conventions, add an entry here so the reasoning survives.

Entries are reverse-chronological. Each entry: date, what changed, **why** (with the failure mode that drove the change when applicable), where the rule lives now.

---

## 2026-05-06 — Incremental mode: add resources to an existing project

**Change:** Workflow step 2 now detects whether `databricks.yml` already exists. If it does, the skill enters **incremental mode** — it only generates new resource YAML files, source code, and test stubs for the requested assets, skipping root folder creation, `databricks.yml`, CI/CD pipelines, and supporting files. A conflict is reported if a resource file for the named asset already exists.

**Why:** Previously every invocation assumed a fresh start scaffold. If a user migrated job_1 and warehouse_1 first, then later wanted to add alert_1, the skill would regenerate the entire project from scratch — overwriting CI/CD files, `databricks.yml`, and other resources. Incremental mode makes "add a resource" a safe, additive operation.

**Where:**
- `SKILL.md` workflow step 2 — new fresh start-vs-incremental branching logic.
- `SKILL.md` steps 3, 5, 6 — tagged *(Fresh start only.)* with incremental-mode exceptions noted.
- `SKILL.md` step 7 — reporting adjusted for incremental mode.
- `SKILL.md` example interactions — added an incremental example alongside the existing fresh start one.

---

## 2026-05-06 — Skeleton sections removed; workflow step 4 uses schema reference

**Change:** Removed the `## Skeleton` sections from all 23 resource `.md` files. Updated workflow step 4 to instruct the agent to map the asset's actual existing attributes against the `## Complete schema reference` rather than copy-pasting a skeleton or the full schema.

**Why:** With the complete schema reference now present in every resource file, the old hand-written skeletons were redundant and could conflict with the authoritative field catalogue. Copying the entire schema verbatim into generated YAML is equally wrong — it produces files full of placeholder values for fields the asset doesn't use. The correct behavior is to include only the fields the asset actually has, validated against the schema.

**Where:**
- All 23 files under `dabs-migrator/resources/` — `## Skeleton` section removed.
- `SKILL.md` workflow step 4 — rewritten to reference the schema as a field catalogue, not a copy-paste template.

---

## 2026-05-06 — Complete schema reference added to all 23 resource files

**Change:** Added a `## Complete schema reference` section to every resource `.md` file, containing the full YAML skeleton with all available fields, types, required/deprecated/preview flags, and short descriptions — all derived from the authoritative `dabs-schema.json` (generated via `databricks bundle schema`).

**Why:** The original hand-written skeletons only covered common fields. During real migrations the agent would omit valid fields or guess at field names/types, producing YAML that failed `bundle validate`. Having the full schema inline means the agent can look up any field without leaving the skill context.

**Where:**
- All 23 files under `dabs-migrator/resources/` — each gained a `## Complete schema reference` section between the existing skeleton/prose and `## What to ask the user`.
- Existing example skeletons, prose, and "What to ask" sections are unchanged.

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
