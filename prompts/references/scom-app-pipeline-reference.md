# SCOM Reusable Workflow Reference

This file is the authoritative reference for SCOM reusable workflow parameters, caller workflow examples, and Jenkins-to-GHA parameter mapping. It is shared across SDM-1 (discovery), SDM-2 (spec), SDM-3 (tasks), SDM-4 (execution), and SDM-5 (validation).

---

## SCOM Java App Pipeline (`scom-app-pipeline.yml`)

**Reusable workflow path:**
```
SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml@main
```

### Inputs

| Input | Type | Required | Default | Description |
|---|---|---|---|---|
| `environment` | string | yes | — | Target environment (dev, qa, staging, prod). Build steps only run for dev. |
| `java-version` | string | no | `'21'` | JDK version |
| `java-distribution` | string | no | `'corretto'` | JDK distribution |
| `container` | string | yes | — | Logical container name (used in concurrency groups) |
| `deploy` | boolean | no | `true` | Enable deployment stages |
| `servers` | string | yes | — | JSON array of server targets (see format below) |
| `maven-deploy` | boolean | no | `false` | Enable Maven artifact publishing to JFrog |
| `oidc-provider-name` | string | yes | — | OIDC provider name in JFrog Artifactory |
| `oidc-audience` | string | yes | — | OIDC audience value in JFrog Artifactory |
| `repository-prefix` | string | yes | — | JFrog Maven repo prefix (e.g., `scom-mvn`, `snet-mvn`) |
| `create-release` | boolean | no | `false` | Create Git tag and GitHub release after deploy |
| `version` | string | no | `''` | App version (required for non-dev environments) |
| `health-check-timeout` | string | no | `'60'` | Seconds to wait for health check |

**Required secrets:** `SSH_PRIVATE_KEY`, `TEAMS_WEBHOOK_URL`

**Required permissions:** `contents: write`, `id-token: write` (plus `checks: write` and `pull-requests: write` if PR-triggered)

**Outputs:** `version` — resolved application version (from Maven build in dev, from input in non-dev)

### Server JSON Format

```json
[
  {
    "host": "hostname",
    "user": "deploy-user",
    "port": "22",
    "path": "/deploy/path",
    "war-name": "app.war",
    "health-check-url": "http://hostname:port/endpoint"
  }
]
```

### Discovery Extraction Guide (SDM-1)

When discovering SCOM Java apps, extract:
1. The `container` name and any `context` path from the shared library call
2. The JDK version (translate Jenkins format: `java-8` → `'8'`, `java-21` → `'21'`)
3. Server hostnames, users, deploy paths, WAR file names, and health check URLs from deployment configuration
4. Whether Maven artifact publishing is enabled
5. The environment promotion chain (dev → qa → staging → prod)

### Reference Caller Workflows

**Single-environment (dev only):**
```yaml
name: Dev Pipeline
on:
  workflow_dispatch:
  pull_request:
    branches: [main]
permissions:
  contents: write
  id-token: write
  checks: write
  pull-requests: write
jobs:
  dev:
    uses: SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml@main
    with:
      environment: dev
      java-version: '21'
      java-distribution: corretto
      container: b2capi
      deploy: true
      maven-deploy: true
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
      health-check-timeout: '60'
      servers: |
        [
          {
            "host": "server1.example.com",
            "user": "deployer",
            "path": "/tmp",
            "war-name": "app.war",
            "health-check-url": "http://server1:8080/actuator/health"
          }
        ]
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

**Multi-environment with promotion chain (dev → qa):**
```yaml
name: CI/CD Pipeline
on:
  workflow_dispatch:
  pull_request:
    branches: [develop]
jobs:
  build-deploy-dev:
    uses: SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml@main
    with:
      environment: dev
      java-version: '8'
      container: serv
      deploy: true
      maven-deploy: true
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
      servers: |
        [
          { "host": "server1", "user": "deployer", "port": "22", "path": "/app/path", "war-name": "serv.war", "health-check-url": "http://server1:8080/serv/version" },
          { "host": "server2", "user": "deployer", "port": "22", "path": "/app/path", "war-name": "serv.war", "health-check-url": "http://server2:8080/serv/version" }
        ]
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}

  deploy-qa:
    needs: build-deploy-dev
    uses: SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml@main
    with:
      environment: qa
      container: serv
      deploy: true
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
      create-release: true
      version: ${{ needs.build-deploy-dev.outputs.version }}
      servers: |
        [...]
    secrets: inherit
```

**Key caller workflow patterns:**
- Dev job builds from source (`environment: dev`) — the reusable workflow runs Maven build only for dev
- Non-dev jobs skip the build and use `version` from the dev job's output
- Non-dev jobs chain via `needs:` to enforce promotion order
- `secrets: inherit` can be used for chained jobs in the same workflow
- Multi-server deployments pass multiple entries in the `servers` JSON array

---

## SCOM Docker/AWS Pipeline (`scom-docker-pipeline.yml`)

**Reusable workflow path:**
```
SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-docker-pipeline.yml@main
```

### Inputs

| Input | Type | Required | Default | Description |
|---|---|---|---|---|
| `environment` | string | yes | — | Target environment (dev, qa, staging, prod). Build steps only run for dev. |
| `container` | string | yes | — | Logical container/app name (ECR repo name, concurrency key) |
| `java-version` | string | no | `'8'` | JDK version |
| `java-distribution` | string | no | `'corretto'` | JDK distribution |
| `dockerfile` | string | no | `'Dockerfile'` | Path to Dockerfile |
| `aws-role-arn` | string | yes | — | IAM role ARN for OIDC authentication to AWS |
| `aws-region` | string | no | `'us-east-1'` | AWS region |
| `ecr-registry` | string | yes | — | ECR registry URL |
| `deploy-ecs` | boolean | no | `true` | Enable ECS/Fargate deployment |
| `ecs-cluster` | string | no | — | ECS cluster name |
| `ecs-service` | string | no | — | ECS service name |
| `ecs-task-definition` | string | no | `'task-definition.json'` | Path to ECS task definition JSON |
| `ecs-container-name` | string | no | — | Container name in task definition |
| `deploy-lambda` | boolean | no | `false` | Enable Lambda function deployment |
| `lambda-function-names` | string (JSON) | no | `'[]'` | JSON array of Lambda function names to update |
| `lambda-maven-profile` | string | no | `'aws-lambda'` | Maven profile for Lambda packaging |
| `maven-deploy` | boolean | no | `false` | Enable Maven artifact publishing to JFrog |
| `oidc-provider-name` | string | yes | — | OIDC provider name for JFrog |
| `oidc-audience` | string | yes | — | OIDC audience for JFrog |
| `repository-prefix` | string | yes | — | JFrog Maven repo prefix |
| `health-check-url` | string | no | — | Health check URL for deployed service |
| `version` | string | no | `''` | App version (required for non-dev environments) |

**Required secrets:** `TEAMS_WEBHOOK_URL` (optional)

**Required permissions:** `contents: write`, `id-token: write`

**Outputs:** `version` (resolved app version), `image-uri` (ECR image URI)

**Composite actions used by the pipeline:**
- `SubaruOfAmerica/devops-cicd-actions-maven-build` — Maven build with JFrog OIDC
- `SubaruOfAmerica/devops-cicd-actions-docker-build` — Docker build and push to ECR
- `SubaruOfAmerica/devops-cicd-actions-aws-deploy` — ECS/Lambda deployment with OIDC

### Discovery Extraction Guide (SDM-1)

When discovering Docker/AWS apps, extract:
1. The Docker image name (usually matches the application/repo name)
2. The JDK version from `pom.xml` (`<java.version>`) or Dockerfile (`FROM openjdk:X`)
3. The Dockerfile location and any build args needed
4. AWS account IDs and regions per environment
5. ECS cluster/service names per environment (if deploying to ECS)
6. Lambda function names (if deploying Lambda functions)
7. Maven profiles for Lambda packaging (e.g., `-Paws-lambda`)
8. The environment promotion chain (dev → qa → preprod → prod)
9. Environment variables and AWS Secrets Manager references
10. Health check endpoints

### Reference Caller Workflows

**Single-environment (Docker/AWS):**
```yaml
name: Dev Pipeline
# on:
#   push:
#     branches: [main]
#   pull_request:
#     branches: [main]
on:
  workflow_dispatch: # Manual-only trigger until migration is validated
permissions:
  contents: write
  id-token: write
jobs:
  dev:
    uses: SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-docker-pipeline.yml@main
    with:
      environment: dev
      container: assetmanager
      java-version: '8'
      aws-role-arn: arn:aws:iam::434206545184:role/github-actions-deploy
      ecr-registry: 434206545184.dkr.ecr.us-east-1.amazonaws.com
      deploy-ecs: true
      ecs-cluster: scom-dev
      ecs-service: assetmanager
      ecs-container-name: assetmanager
      deploy-lambda: true
      lambda-function-names: '["SQSMessageHandlerStore", "SQSMessageHandlerRetrieve", "SQSMessageHandlerNotify", "SQSMessageHandlerDelete"]'
      lambda-maven-profile: aws-lambda
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
      health-check-url: http://assetmanager-dev:8088/actuator/health
    secrets:
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

**Multi-environment with promotion chain (Docker/AWS):**
```yaml
name: CI/CD Pipeline
on:
  workflow_dispatch:
jobs:
  build-deploy-dev:
    uses: SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-docker-pipeline.yml@main
    with:
      environment: dev
      container: assetmanager
      java-version: '8'
      aws-role-arn: arn:aws:iam::434206545184:role/github-actions-deploy
      ecr-registry: 434206545184.dkr.ecr.us-east-1.amazonaws.com
      deploy-ecs: true
      ecs-cluster: scom-dev
      ecs-service: assetmanager
      ecs-container-name: assetmanager
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
    secrets:
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}

  deploy-prod:
    needs: build-deploy-dev
    uses: SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-docker-pipeline.yml@main
    with:
      environment: prod
      container: assetmanager
      aws-role-arn: arn:aws:iam::926569254619:role/github-actions-deploy
      ecr-registry: 926569254619.dkr.ecr.us-east-1.amazonaws.com
      deploy-ecs: true
      ecs-cluster: scom-prod
      ecs-service: assetmanager
      ecs-container-name: assetmanager
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
      version: ${{ needs.build-deploy-dev.outputs.version }}
    secrets: inherit
```

**Key Docker/AWS caller workflow patterns:**
- Dev job builds from source, builds Docker image, pushes to ECR — the reusable workflow runs Maven build and Docker build only for dev
- Non-dev jobs skip the build and use `version` from the dev job's output to resolve the image URI
- Non-dev jobs chain via `needs:` to enforce promotion order
- Each environment may use a different AWS account and IAM role ARN
- Lambda function names are passed as a JSON array for batch updates
- `secrets: inherit` can be used for chained jobs in the same workflow

### Reference ECS Task Definition (`task-definition.json`)

```json
{
  "family": "assetmanager",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::ACCOUNT_ID:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::ACCOUNT_ID:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "assetmanager",
      "image": "ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/assetmanager:latest",
      "portMappings": [
        {
          "containerPort": 8088,
          "protocol": "tcp"
        }
      ],
      "environment": [
        { "name": "SPRING_PROFILES_ACTIVE", "value": "dev" }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/assetmanager",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8088/actuator/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

---

## Jenkins-to-Reusable-Workflow Parameter Mapping

| Jenkins `scomAppPipeline()` Param | Reusable Workflow Input | Transformation |
|---|---|---|
| `container: 'name'` | `container: name` | Direct mapping |
| `jdk: 'java-8'` | `java-version: '8'` | Strip `java-` prefix |
| `deploy: true/false` | `deploy: true/false` | Direct mapping |
| `context: '/path'` | Part of `health-check-url` in `servers` JSON | Combine with host and port |
| `enabled: true` | `deploy: true` | Rename |
| (not in Jenkins) | `oidc-provider-name`, `oidc-audience` | New — ask user for OIDC config |
| (not in Jenkins) | `repository-prefix` | New — ask user for JFrog repo prefix |
| (not in Jenkins) | `servers` JSON | New — ask user for server details (host, user, path, war-name, health-check-url) |
