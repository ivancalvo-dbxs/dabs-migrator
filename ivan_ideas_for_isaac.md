# Idea for the project

* Create a single workspace agent skill for Databricks as listed in: https://docs.databricks.com/aws/en/genie-code/skills#create-a-skill
* The skill is going to generate what is know as a Databricks Asset Bundles project code structure.
* The input are Databricks resources names.
* The output is going to be DABs-ready files and folder structures.


## Input

The user is going to specify the names of the Databricks Assets or Objects, the supported list can be obtained from the following table: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#supported-resources

## Output

A repository with DABs folders and files. Here are examples of GitHub folders with a well-defined DABs repo structure:

* https://github.com/databricks-solutions/databricks-dab-examples/tree/main/sts-dabs-demo
* https://github.com/databricks-solutions/databricks-dab-examples/tree/main/flights/flights-simple
* https://github.com/databricks-solutions/databricks-dab-examples/tree/main/flights/flights-advanced

# Example of a user interaction with the skill

## Input

I want to migrate @my_job_1 and @my_pipeline_1 to DABs, generate the project and for the CI/CD tool use Github Actions.

## Output

The output is a parent folder with subfolders and files for the Databricks DABs project:

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

You get more context, details and ideas from the markdown file on: databricks-project-structure.yml

## Consideration when generating the CI/CD files.

The CI/CD files should follow the next actions:

* Install the Databricks CLI.
* Run the DABs validation command using —output json flag.
* Run the DABs deploy command.

More information about the commands and examples can be found here: 
	* https://docs.databricks.com/aws/en/dev-tools/bundles/jobs-tutorial

### Never do

* Never do updates or work related to databricks folders. Ban all “databricks repos” commands.

# The Skill

* Skill name is DABs migrator.
* Within the skill, create a dedicated skill for each supported DABs resource: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#supported-resources
  * i.e:
    * dabs-migrator (skill-folder)
      * SKILL.md
      * resources (resources folder)
        * jobs.md
        * pipelines.md
        * ...

