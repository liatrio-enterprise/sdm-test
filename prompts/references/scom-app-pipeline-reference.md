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
| `java-distribution` | string | no | `'corretto'` | JDK distribution (e.g., corretto, temurin, zulu) |
| `container` | string | yes | — | Logical container name (used by env-config to resolve server configuration) |
| `deployment-mode` | string | no | `'blue-green'` | Deployment strategy: `blue-green` (zero-downtime with site-status switching) or `single` (sequential to all servers) |
| `oidc-provider-name` | string | yes | — | OIDC provider name configured in JFrog Artifactory |
| `oidc-audience` | string | yes | — | OIDC audience value configured in JFrog Artifactory |
| `repository-prefix` | string | yes | — | JFrog Maven repo prefix (e.g., `scom-mvn`, `snet-mvn`). Configures both `{prefix}-release` and `{prefix}-snapshot` repos. |
| `version` | string | no | `''` | App version (required for non-dev environments) |
| `health-check-url` | string | yes | — | Health check URL curled via SSH on the target server after deploy (e.g., `http://b2capi.qa.subaru.com/b2capi/actuator/health`) |
| `deploy-path` | string | yes | — | Absolute deploy path on the server (e.g., `/app/home/embedded_tomcat/b2capi`) |

**Required secrets:** `SSH_PRIVATE_KEY`, `TEAMS_WEBHOOK_URL`

**Required permissions:** `contents: write`, `id-token: write` (plus `checks: write` and `pull-requests: write` if PR-triggered)

**Outputs:**
- `version` — resolved application version (from Maven build in dev, from input in non-dev)
- `active-host` — hostname of the active server (from env-config via deploy)
- `standby-host` — hostname of the standby server (from env-config via deploy)

> **Server resolution:** The reusable workflow resolves server hostnames, users, and ports dynamically via the `env-config` action using the `container` and `environment` inputs. Callers do not pass server details — they only provide `container`, `health-check-url`, and `deploy-path`.

### Discovery Extraction Guide (SDM-1)

When discovering SCOM Java apps, extract:
1. The `container` name from the shared library call (e.g., `container: 'b2capi'`)
2. The JDK version (translate Jenkins format: `java-8` → `'8'`, `java-21` → `'21'`)
3. The `deploy-path` — absolute path on the server where the application is deployed
4. The `health-check-url` — URL used to verify the application is running after deploy
5. The `deployment-mode` preference — `blue-green` (default) or `single`
6. The environment promotion chain (dev → qa → staging → prod)
7. OIDC and JFrog repository configuration (`oidc-provider-name`, `oidc-audience`, `repository-prefix`)

### Reference Caller Workflows

Applications use a **two-file pattern**:
1. **`dev-qa-pipeline.yml`** — Handles dev (builds from source) and qa (promotes the dev build). Dev and QA deploy to the same server pair.
2. **`prod-pipeline.yml`** — Triggers on dev/qa workflow success via `workflow_run`, gated by environment protection rules for manual approval. Resolves version from the completed dev/qa run.

**`dev-qa-pipeline.yml` (build + promote):**
```yaml
name: Dev/QA Pipeline
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
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
      health-check-url: http://b2capi.qa.subaru.com/b2capi/actuator/health
      deploy-path: /app/home/embedded_tomcat/b2capi
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SCOM_CICD_DEV_SSH }}
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}

  qa:
    needs: dev
    uses: SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml@main
    with:
      environment: qa
      java-version: '21'
      java-distribution: corretto
      container: b2capi
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
      version: ${{ needs.dev.outputs.version }}
      health-check-url: http://b2capi.qa.subaru.com/b2capi/actuator/health
      deploy-path: /app/home/embedded_tomcat/b2capi
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SCOM_CICD_DEV_SSH }}
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

**`prod-pipeline.yml` (deploy on dev/qa success, gated by manual approval):**
```yaml
name: Prod Pipeline
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to deploy (e.g., 1.2.3). Leave empty to use version from latest Dev/QA run.'
        required: false
        type: string
  workflow_run:
    workflows: ["Dev/QA Pipeline"]
    types: [completed]
    branches: [main]
permissions:
  contents: write
  id-token: write
  actions: read
jobs:
  resolve-version:
    if: >-
      (github.event_name == 'workflow_dispatch') ||
      (github.event_name == 'workflow_run' && github.event.workflow_run.conclusion == 'success')
    runs-on: LandingZones
    outputs:
      version: ${{ steps.version.outputs.version }}
    steps:
      - name: Resolve version
        id: version
        env:
          GH_TOKEN: ${{ github.token }}
        shell: bash
        run: |
          if [ "${{ github.event_name }}" = "workflow_dispatch" ] && [ -n "${{ inputs.version }}" ]; then
            echo "version=${{ inputs.version }}" >> "$GITHUB_OUTPUT"
          else
            # Get version from the completed Dev/QA workflow run
            RUN_ID="${{ github.event.workflow_run.id }}"
            if [ -z "$RUN_ID" ]; then
              # workflow_dispatch without version — find latest successful Dev/QA run
              RUN_ID=$(gh run list --workflow="Dev/QA Pipeline" --status=success --limit=1 --json databaseId --jq '.[0].databaseId')
            fi
            # Extract version from the dev job outputs
            VERSION=$(gh run view "$RUN_ID" --json jobs --jq '.jobs[] | select(.name | startswith("dev")) | .outputs.version' 2>/dev/null || true)
            if [ -z "$VERSION" ]; then
              echo "::error::Could not resolve version from Dev/QA run $RUN_ID"
              exit 1
            fi
            echo "version=$VERSION" >> "$GITHUB_OUTPUT"
          fi

  prod:
    needs: resolve-version
    uses: SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml@main
    with:
      environment: prod
      java-version: '21'
      java-distribution: corretto
      container: b2capi
      oidc-provider-name: soa-scom-github
      oidc-audience: soa-scom-github
      repository-prefix: scom-mvn
      version: ${{ needs.resolve-version.outputs.version }}
      health-check-url: http://b2capi.prod.subaru.com/b2capi/actuator/health
      deploy-path: /app/home/embedded_tomcat/b2capi
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SCOM_CICD_PROD_SSH }}
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

**Key caller workflow patterns:**
- Dev job builds from source (`environment: dev`) — the reusable workflow runs Maven build only for dev
- QA job chains after dev via `needs:` and passes `version: ${{ needs.dev.outputs.version }}`
- Dev and QA share the same server pair; the env-config action resolves the correct servers per environment
- Prod is in a separate workflow file — triggers on dev/qa success via `workflow_run`, with environment protection rules for manual approval
- Prod also supports manual `workflow_dispatch` with an optional version override
- The reusable workflow defaults to blue-green deployment; callers can override with `deployment-mode: single`
- `health-check-url` and `deploy-path` are required for all environments
- Server hostnames, users, and ports are resolved automatically by the env-config action — callers only pass `container` and `environment`

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
| `context: '/path'` | Part of `health-check-url` | Combine with hostname and port to form full health check URL |
| `enabled: true` | (no equivalent — always deploys) | Removed; deployment always occurs |
| `deploy: true/false` | (removed) | No longer an input; deployment always occurs |
| (Jenkins server config) | `deploy-path` | Extract from Jenkins deployment configuration |
| (Jenkins health check) | `health-check-url` | Extract from Jenkins deployment configuration |
| (not in Jenkins) | `deployment-mode` | New — defaults to `blue-green`; set to `single` if needed |
| (not in Jenkins) | `oidc-provider-name`, `oidc-audience` | New — ask user for OIDC config |
| (not in Jenkins) | `repository-prefix` | New — ask user for JFrog repo prefix |
