# Resource: `external_locations`

Unity Catalog external locations — pointers to cloud storage paths secured by a storage credential.

Docs: https://docs.databricks.com/aws/en/dev-tools/bundles/resources#external_location

## Skeleton

```yaml
resources:
  external_locations:
    {{ location_name }}:
      name: {{ location_name }}
      url: s3://my-bucket/path
      credential_name: {{ credential_name }}
      comment: "External location managed by ${bundle.name}"
      grants:
        - principal: data-engineers
          privileges: [READ_FILES, WRITE_FILES, CREATE_EXTERNAL_TABLE]
```

## What to ask the user

- Cloud storage URL (s3://, abfss://, gs://)?
- Existing storage credential name?
