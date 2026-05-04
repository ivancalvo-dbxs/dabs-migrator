# Resource: `experiments`

MLflow experiments.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#experiment

## Skeleton

```yaml
resources:
  experiments:
    {{ experiment_name }}:
      name: /Workspace/Shared/experiments/${bundle.name}/{{ experiment_name }}
      description: "Experiment managed by ${bundle.name}"
      tags:
        - key: project
          value: ${bundle.name}

      permissions:
        - level: CAN_EDIT
          group_name: ml-engineers
```

## What to ask the user

- Workspace path for the experiment?
- Tags / project metadata?
