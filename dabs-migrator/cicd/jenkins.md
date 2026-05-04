# CI/CD: Jenkins

Declarative `Jenkinsfile` at repo root. Assumes Jenkins agents have `curl` and `bash`.

## Required Jenkins credentials

| ID | Type |
|---|---|
| `databricks-host-staging` | Secret text |
| `databricks-host-prod` | Secret text |
| `databricks-token` | Secret text |

## `Jenkinsfile`

```groovy
pipeline {
  agent any

  environment {
    DATABRICKS_TOKEN = credentials('databricks-token')
  }

  stages {
    stage('Install Databricks CLI') {
      steps {
        sh 'curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh'
      }
    }

    stage('Validate') {
      environment {
        DATABRICKS_HOST = credentials('databricks-host-staging')
      }
      steps {
        sh 'databricks bundle validate --output json | tee bundle-validate.json'
        archiveArtifacts artifacts: 'bundle-validate.json', fingerprint: true
      }
    }

    stage('Deploy staging') {
      when { branch 'main' }
      environment {
        DATABRICKS_HOST = credentials('databricks-host-staging')
      }
      steps {
        sh 'databricks bundle deploy -t staging'
      }
    }

    stage('Deploy prod') {
      when { buildingTag() }
      environment {
        DATABRICKS_HOST = credentials('databricks-host-prod')
      }
      input {
        message 'Deploy to prod?'
        ok 'Deploy'
      }
      steps {
        sh 'databricks bundle validate --output json'
        sh 'databricks bundle deploy -t prod'
      }
    }
  }
}
```

## Notes

- The `input` block on the prod stage gates deployment behind a manual approval.
- Run on a tag build (`buildingTag()`) — configure your multibranch/MultiPipeline to discover tags.
