# Resource: `model_serving_endpoints`

REST endpoints for serving registered models.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#model_serving_endpoint

## Skeleton

```yaml
resources:
  model_serving_endpoints:
    {{ endpoint_name }}:
      name: {{ endpoint_name }}
      config:
        served_entities:
          - name: ${var.catalog}-{{ model_name }}-1
            entity_name: ${var.catalog}.ml_models.{{ model_name }}
            entity_version: "1"
            workload_size: Small
            scale_to_zero_enabled: true
        traffic_config:
          routes:
            - served_model_name: ${var.catalog}-{{ model_name }}-1
              traffic_percentage: 100

      tags:
        - key: project
          value: ${bundle.name}

      permissions:
        - level: CAN_QUERY
          group_name: api-consumers
```

## What to ask the user

- Which registered model + version(s)?
- Workload size and scale-to-zero?
- Traffic split if multiple models?
