---
name: SDM-1-discovery-assessment
description: "Audit existing Jenkins pipelines (Jenkinsfile), plugins, credentials, shared libraries, and infrastructure for GitHub Actions migration. Also supports greenfield CI/CD assessment for applications without existing pipelines. Use when the user needs a discovery assessment for GitHub Actions."
tags:
  - migration
  - assessment
  - jenkins
  - ci-cd
arguments: []
meta:
  category: spec-driven-migration
  allowed-tools: Glob, Grep, LS, Read, WebFetch, WebSearch
---

# Discovery and Assessment

## Context Marker

Always begin your response with all active emoji markers, in the order they were introduced.

Format:  "<marker1><marker2><marker3>\n<response>"

The marker for this instruction is:  SDM1️⃣

## You are here in the workflow

**Step 1 of 5.** Produce a comprehensive pipeline/application inventory that becomes the factual foundation for the entire migration.

**Depends on:** User-provided Jenkinsfile or application repository → **Produces for:** SDM-2 (migration spec)

### Output Type Rule

The migration output type depends on the source — classify early so the discovery report frames the right migration strategy:

- **SCOM Java application** (Jenkinsfile calling `scomAppPipeline()` or similar SCOM shared library) → **Thin caller workflow** that invokes the existing `scom-app-pipeline.yml` reusable workflow in `SubaruOfAmerica/devops-cicd-workflows`. The reusable workflow already handles build, deploy, and notification logic — the migration task is to map Jenkins parameters to reusable workflow inputs. **This is the most common migration type.**
- **Jenkinsfile without shared libraries** → GitHub Actions workflow (`.github/workflows/<name>.yml`)
- **Jenkinsfile calling non-SCOM shared libraries** (`@Library`) → Application workflow + reusable workflow(s) via `workflow_call`. Ask the user where the reusable workflow should live (app repo, shared-workflows repo, or org-level repo)
- **Standalone shared library** (`vars/*.groovy`) → Reusable workflow; prompt for target repository
- **SCOM Docker/AWS application** (no Jenkinsfile, or Jenkinsfile with Docker/AWS deployment — application uses Dockerfile, deploys to AWS ECS/Lambda) → **Thin caller workflow** that invokes the existing `scom-docker-pipeline.yml` reusable workflow in `SubaruOfAmerica/devops-cicd-workflows`. The reusable workflow handles Maven build, Docker image build/push to ECR, and AWS deployment (ECS and/or Lambda). The migration task is to map application configuration to reusable workflow inputs.

### SCOM Reusable Workflow Reference

When the migration target is the SCOM reusable workflow, discovery should focus on extracting parameters that map to the reusable workflow's inputs. For the full parameter table and caller workflow examples, read `prompts/references/scom-app-pipeline-reference.md` (section: "SCOM Java App Pipeline" → "Discovery Extraction Guide").

The reusable workflow lives at:
```
SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml@main
```

### SCOM Docker Pipeline Reference

When the target is a Docker/AWS application, discovery should focus on extracting parameters that map to the Docker pipeline's inputs. For the full parameter table, composite actions, and discovery extraction guide, read `prompts/references/scom-app-pipeline-reference.md` (section: "SCOM Docker/AWS Pipeline" → "Discovery Extraction Guide").

The reusable workflow lives at:
```
SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-docker-pipeline.yml@main
```

## Your Role

You are a **Senior DevOps Engineer and CI/CD Migration Specialist** with deep expertise in Jenkins architecture, pipeline configuration, migration planning, and cloud-native deployment patterns (Docker, AWS ECS, Lambda). You understand Jenkins internals — declarative and scripted pipelines, shared libraries, plugin ecosystems, credential management, and agent topologies. You also understand modern container-based deployments and can assess applications that need greenfield CI/CD pipelines. Your job is to produce a thorough, accurate inventory that will become the foundation for migration planning.

## Goal

Produce a comprehensive inventory for the pipeline(s) or application(s) in scope. This inventory becomes the primary input for `/SDM-2-generate-migration-spec`. The discovery must be thorough enough that the migration spec can be written without referring back to the original source files.

There are two discovery modes:
1. **Migration mode** (default): The user has an existing Jenkinsfile or Jenkins pipeline to migrate. Audit the Jenkins configuration.
2. **Greenfield mode**: The user has an application with no CI/CD pipeline. Audit the application's build system, Dockerfile, deployment artifacts, and infrastructure to produce a discovery report.

If the user did not provide a Jenkinsfile or reference to their Jenkins pipeline configuration, check if the application has a Dockerfile, pom.xml, or other build system artifacts. If so, proceed in greenfield mode. Otherwise, ask the user to provide their pipeline configuration or confirm greenfield assessment.

## Discovery Process Overview

1. **Repository Details** — Detect the repository's default branch and basic Git metadata
2. **Locate Jenkinsfiles** — Scan the repository for all pipeline definitions
3. **Pipeline Classification** — Categorize each pipeline by type, complexity, and triggers
4. **Plugin Inventory** — Detect all plugin usage from pipeline syntax
5. **Credentials Audit** — Catalog all credential references by type (never extract values)
6. **Agent/Node Assessment** — Map agent configurations to runner strategies
7. **Shared Library Analysis** — Document library usage, roles, and complexity
8. **Integration Points** — Identify all external system connections
9. **Scope Assessment** — Evaluate if the migration is appropriately sized

### Greenfield Discovery Process (when no Jenkinsfile exists)

If no Jenkinsfile is found and the application has build artifacts (pom.xml, Dockerfile, etc.), switch to greenfield discovery:

1. **Repository Details** — Detect the repository's default branch and basic Git metadata (same as above)
2. **Build System Analysis** — Examine pom.xml/build.gradle for dependencies, plugins, profiles, packaging type
3. **Dockerfile Analysis** — Examine Dockerfile for base image, exposed ports, build stages, profiles
4. **Deployment Target Assessment** — Identify deployment targets from application config (AWS resources, environment variables, secrets)
5. **Credentials & Environment Audit** — Catalog all environment variables, AWS Secrets Manager references, config files per profile
6. **Integration Points** — Identify external services (databases, message queues, APIs, cloud services)
7. **Infrastructure Mapping** — Map AWS accounts, regions, and resources per environment
8. **Scope Assessment** — Evaluate if this is appropriate for the Docker pipeline workflow

## Step 0: Repository Details

Before analyzing pipelines or build systems, detect the repository's default branch. This is needed for workflow trigger configuration — dev/QA workflows trigger on pull requests to the default branch, while prod workflows trigger on push to `main`.

**Detection method:**
```bash
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@'
```

If the above fails (e.g., `origin/HEAD` is not set), fall back to:
1. Check `git branch -r` for `origin/HEAD -> origin/<branch>`
2. Look at the current checked-out branch
3. Ask the user

**Record in discovery report:**

| Attribute | Value |
|---|---|
| **Default Branch** | `<detected branch name>` |

> **Why this matters:** Not all repositories use `main` as their default branch. Some use `develop` or other branch names. The dev/QA caller workflow must target the actual default branch in its `pull_request` trigger. The prod workflow always targets `main`. For repos where the default branch is not `main`, a release merge into `main` triggers the prod deploy.

## Step 1: Locate Jenkinsfiles

Scan the repository for all Jenkins pipeline definitions:

**File patterns to search:**

- `Jenkinsfile` (root and subdirectories)
- `Jenkinsfile.*` (e.g., `Jenkinsfile.deploy`, `Jenkinsfile.release`)
- `*.jenkinsfile`
- Files in `jenkins/` or `.jenkins/` directories
- `@Library` references pointing to external shared library repositories

**For each file found, record:**

- File path relative to repository root
- File size and last modified date (from git history)
- Whether it imports shared libraries

### Shared Library Wrapper Detection

After locating Jenkinsfiles, check whether any are **thin wrappers** — Jenkinsfiles that contain little to no inline pipeline logic and instead delegate entirely to a shared library function. Common indicators:

- The entire Jenkinsfile is a single function call (e.g., `scomAppPipeline(...)`, `orgPipeline(...)`, `buildAndDeploy(...)`)
- No `pipeline {}` block, no `stage {}` definitions, no `steps {}` blocks
- The file is very short (under ~10 lines of actual code, excluding comments)
- Parameters are passed as a map to a top-level function rather than declared inline

#### SCOM Shared Library Pattern (common case)

If the Jenkinsfile calls `scomAppPipeline(...)`, this is an **SCOM application pipeline** and the migration target is a thin caller workflow invoking the existing `scom-app-pipeline.yml` reusable workflow. In this case:

1. **Do NOT request the shared library source.** The shared library logic has already been migrated into the reusable workflow at `SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml`.
2. **Extract the parameters** passed to `scomAppPipeline()` — these map directly to the reusable workflow inputs (see SCOM Reusable Workflow Reference above).
3. **Proceed with discovery** using the parameter mapping approach, supplemented by the repo's `pom.xml` for artifact/version details. Note: server hostnames and users are resolved dynamically by the env-config action — discovery should focus on extracting `container`, `deploy-path`, and `health-check-url` instead of individual server details.

Example SCOM Jenkinsfile:
```groovy
scomAppPipeline(enabled: true, container: 'serv', context: '/serv', deploy: true, jdk: 'java-8')
```

This maps to a caller workflow with: `container: serv`, `java-version: '8'`. The `deploy` and `context` Jenkins parameters are no longer direct inputs — `context` is incorporated into `health-check-url`, and deployment always occurs.

#### Non-SCOM Wrapper Pattern

If a non-SCOM wrapper pattern is detected, STOP and inform the user:

> This Jenkinsfile delegates its entire pipeline logic to a shared library function (`<functionName>`). The actual stages, credentials, plugins, and deployment logic live in the shared library source code, not in this file. To complete discovery, I need access to the shared library.
>
> Do you have the shared library repository available locally? If so, please provide the path (e.g., `~/repos/jenkins-shared-lib/vars/`).

**Do not proceed past Step 1 until shared library source is available.** Without it, the discovery report will be incomplete — the Jenkinsfile alone does not contain the information needed for Steps 2–7.

### Greenfield Detection (No Jenkinsfile)

If no Jenkinsfile is found in the repository, check for:

1. **Dockerfile** — Indicates a containerized application
2. **pom.xml / build.gradle** — Indicates a Java/JVM application  
3. **AWS configuration** — Environment variables referencing AWS services (`AWS_REGION`, `scomstorageKeyId`, etc.), SDK dependencies in pom.xml (`aws-java-sdk-*`, `software.amazon.awssdk.*`)
4. **Lambda handlers** — Classes implementing `RequestHandler<SQSEvent, Void>` or similar AWS Lambda interfaces
5. **Application config** — Spring Boot `application.properties`/`application.yml` with profile-specific configs

If Docker + AWS indicators are found, this is an **SCOM Docker/AWS application** and the target is the `scom-docker-pipeline.yml` reusable workflow. Proceed with greenfield discovery using the build system analysis steps instead of Jenkins pipeline analysis.

#### Greenfield Discovery Report Structure

The discovery report for greenfield assessments follows the same output format but replaces Jenkins-specific sections:

| Standard Section | Greenfield Equivalent |
|---|---|
| Pipeline Inventory | Build System Inventory (pom.xml, Dockerfile, Maven profiles) |
| Plugin Dependency Matrix | Dependency Inventory (Maven dependencies, Docker base images) |
| Credentials Catalog | Environment Variables & Secrets Catalog |
| Agent/Runner Mapping | Infrastructure Mapping (AWS accounts, regions, resources) |
| Shared Library Catalog | N/A (or: Shared dependencies/custom libraries) |
| Integration Points | Same (external services, deployment targets, notification channels) |

## Step 2: Pipeline Classification

For each Jenkinsfile found, classify:

### Pipeline Type

| Attribute | Value |
|---|---|
| **Syntax** | Declarative / Scripted / Mixed |
| **Complexity** | Simple (linear) / Moderate (parallel, conditional) / Complex (matrix, dynamic stages) |
| **Trigger(s)** | SCM polling / Cron / Webhook / Upstream / Manual (`input`) |
| **Parameters** | List all `parameters {}` block entries with types and defaults |
| **Stage Count** | Number of pipeline stages |
| **Parallel Stages** | Yes/No — list parallel stage groups |
| **Matrix Build Candidates** | Yes/No — see Matrix Build Opportunity Analysis below |
| **Conditional Logic** | `when` blocks, `if/else` (scripted), branch-based filtering |
| **Post-Conditions** | `post { always/success/failure/unstable/cleanup }` actions |
| **Timeout/Retry** | Any `timeout` or `retry` configurations |

### Matrix Build Opportunity Analysis

After classifying each pipeline, analyze it for **GitHub Actions matrix build opportunities**. Jenkins pipelines often implement multi-version, multi-platform, or multi-environment builds using patterns that map directly to `strategy.matrix` in GitHub Actions.

**Detection patterns — look for any of the following:**

| Jenkins Pattern | Example Syntax | Matrix Opportunity |
|---|---|---|
| Parallel stages with identical structure | `parallel { stage('Node 16') { ... } stage('Node 18') { ... } }` | `matrix: { node-version: [16, 18] }` |
| Loop-generated stages | `for (version in versions) { stage("Build ${version}") { ... } }` | Matrix with version dimension |
| Parameterized multi-value builds | `choice(choices: ['dev', 'staging', 'prod'])` with same logic per choice | Matrix with environment dimension |
| Multi-platform builds | Separate stages for `linux`, `windows`, `macos` agents | `matrix: { os: [ubuntu-latest, windows-latest, macos-latest] }` |
| Multi-architecture builds | Docker buildx or separate stages for `amd64`, `arm64` | Matrix with architecture dimension |
| Repeated stages differing only by a variable | Stages like `Test-Java11`, `Test-Java17` with same steps | Matrix with the varying variable as dimension |
| Scripted `Map` iteration | `def configs = ['api': [...], 'web': [...]]` iterated in parallel | Matrix with component dimension |

**For each matrix candidate found, record:**

| Candidate | Jenkins Implementation | Suggested Matrix Dimensions | Estimated Combinations | Notes |
|---|---|---|---|---|
| [description] | [parallel stages / loop / parameter] | [e.g., `node-version: [16, 18, 20]`] | [count] | [fail-fast behavior, exclude combos, etc.] |

**Also assess:**

- **Fail-fast behavior**: Does the Jenkins pipeline stop all parallel branches on first failure? Map to `fail-fast: true/false`
- **Excludes**: Are there combinations that should be skipped? Map to `matrix.exclude`
- **Includes**: Are there one-off combinations with additional variables? Map to `matrix.include`
- **Max parallel**: Does Jenkins limit concurrent parallel branches? Map to `max-parallel:`

**If no matrix candidates are found**, explicitly state: "No matrix build opportunities identified — pipeline stages are sequential or structurally unique."

## Step 3: Plugin Inventory

Detect plugin usage from Jenkinsfile syntax. Create a comprehensive inventory:

| Jenkins Plugin/Feature | Detected Syntax | Purpose | Usage Count |
|---|---|---|---|
| Credentials Binding | `withCredentials([...])` | Secret injection into build steps | |
| Docker Pipeline | `docker.build()`, `docker.image()`, `agent { docker {} }` | Container-based builds | |
| Kubernetes | `agent { kubernetes {} }`, `podTemplate` | K8s-based build agents | |
| Pipeline Utility Steps | `readJSON`, `readYAML`, `readFile`, `writeFile` | File manipulation | |
| JUnit | `junit '**/test-results/*.xml'` | Test result publishing | |
| Cobertura/JaCoCo | `cobertura`, `jacoco` | Code coverage reporting | |
| HTML Publisher | `publishHTML` | HTML report publishing | |
| Slack Notification | `slackSend` | Slack messaging | |
| Email Extension | `emailext` | Email notifications | |
| SonarQube Scanner | `withSonarQubeEnv`, `waitForQualityGate` | Code quality analysis | |
| Artifactory/Nexus | `rtUpload`, `rtDownload`, `nexusArtifactUploader` | Artifact management | |
| AWS Steps | `withAWS`, `s3Upload`, `s3Download` | AWS integration | |
| Azure CLI | `azureCLI` | Azure integration | |
| GCloud SDK | `withGCloud` | GCP integration | |
| SSH Agent | `sshagent` | SSH key injection | |
| HTTP Request | `httpRequest` | HTTP API calls | |
| Lockable Resources | `lock` | Concurrency control | |
| Build Discarder | `buildDiscarder`, `logRotator` | Build history management | |
| Parameterized Trigger | `build job:` | Downstream job triggering | |
| Workspace Cleanup | `cleanWs()`, `deleteDir()` | Workspace management | |
| Warnings Next Gen | `recordIssues` | Static analysis reporting | |

**Important:** Also look for less obvious plugin usage — any method call in a Jenkinsfile that is not a core Pipeline step likely comes from a plugin.

## Step 4: Credentials Audit

> **Note:** Credentials are inventoried here for awareness and post-migration tracking. The actual secret configuration in GitHub Actions is a post-migration activity — not part of the core pipeline output.

Document all credentials referenced in the pipeline(s). **Never extract or display actual secret values.**

| Credential ID | Type | Usage Context | Stage(s) Used In |
|---|---|---|---|
| [id from `credentialsId`] | `usernamePassword` | Docker registry login | Build, Push |
| [id from `credentialsId`] | `string` | API token for deployment | Deploy |
| [id from `credentialsId`] | `sshUserPrivateKey` | SSH access to servers | Deploy |
| [id from `credentialsId`] | `file` | Kubeconfig for cluster access | Deploy |
| [id from `credentialsId`] | `certificate` | TLS certificate for signing | Build |

**Detection patterns:**

- `withCredentials([usernamePassword(...)])` / `withCredentials([string(...)])` / `withCredentials([sshUserPrivateKey(...)])`
- `credentials('id')` in `environment {}` blocks
- `withAWS(credentials: 'id')` / `withAWS(roleAccount: '...')`
- `docker.withRegistry('url', 'credentialId')`
- Any `credentialsId` parameter in plugin steps

**CRITICAL: Never attempt to read, display, or extract actual secret values. Only document credential IDs, types, and usage context.**

## Step 5: Agent/Node Assessment

Map all `agent` configurations to potential GitHub Actions runner strategies:

| Jenkins Agent Config | Current Purpose | Suggested GHA Runner | Notes |
|---|---|---|---|
| `agent any` | Run on any available node | `runs-on: ubuntu-latest` | Simplest migration |
| `agent none` | No default agent (per-stage) | Per-job `runs-on:` | Each stage becomes a separate job |
| `agent { label 'linux' }` | Specific node label | `runs-on: [self-hosted, linux]` | Requires self-hosted runner setup |
| `agent { docker { image 'node:18' } }` | Docker container agent | `container: image: node:18` | GHA container job |
| `agent { kubernetes { ... } }` | Kubernetes pod agent | Actions Runner Controller (ARC) | Significant infrastructure setup |
| `agent { dockerfile true }` | Build from repo Dockerfile | `container:` with prior build step | Two-step process in GHA |

**Also document:**

- CPU/memory requirements if specified in agent configs
- Network access requirements (VPN, private subnets)
- Tool requirements installed on agents (specific JDK versions, build tools, etc.)
- Whether agents are ephemeral or persistent

## Step 6: Shared Library Analysis

For each `@Library` annotation detected:

| Library | Import Statement | Functions/Vars Used | Complexity | Migration Strategy |
|---|---|---|---|---|
| [name] | `@Library('name') _` | `buildApp()`, `deployToK8s()` | [Simple/Moderate/Complex] | [Composite action / Reusable workflow / Inline script] |

**For each shared library function used:**

- **Purpose:** What does this function do?
- **Inputs:** What parameters does it accept?
- **Side Effects:** Does it modify workspace, call external APIs, manage credentials?
- **Dependencies:** Does it depend on other library functions or specific plugins?
- **Source Available:** Can you read the library source code? (Check `vars/` and `src/` directories if accessible)

**Migration strategy decision tree:**

- Simple utility (string manipulation, file operations) → **Inline script or shell step**
- Build/test helper (compile, test, package) → **Composite action**
- Complex multi-stage orchestration (build + deploy + notify) → **Reusable workflow**
- Organization-wide standard (enforced compliance, audit) → **Reusable workflow in dedicated repo**

## Step 7: Integration Points

Identify all external systems the pipeline connects to:

### External Services

| Service | Purpose | Connection Method | Authentication |
|---|---|---|---|
| [e.g., SonarQube] | Code quality | API call via plugin | Token credential |
| [e.g., Artifactory] | Artifact storage | Plugin / REST API | Username/password |
| [e.g., AWS ECR] | Container registry | Docker push | AWS credentials |

### Deployment Targets

| Target | Environment | Method | Credential Type |
|---|---|---|---|
| [e.g., K8s cluster] | Production | kubectl apply | Kubeconfig file |
| [e.g., EC2 instances] | Staging | SSH / SSM | SSH key / IAM role |

### Notification Channels

| Channel | Trigger | Method |
|---|---|---|
| [e.g., Slack #deploys] | Success/Failure | `slackSend` plugin |
| [e.g., team@company.com] | Failure only | `emailext` plugin |

### Approval Gates

| Gate | Stage | Approvers | Timeout |
|---|---|---|---|
| [e.g., Production deploy] | Deploy-Prod | [team/individuals] | [timeout duration] |

### CLI-to-Action Opportunities

Scan all `sh` / `bat` steps for CLI tool usage where a major platform vendor provides an official GitHub Action. These represent opportunities to replace raw shell commands with purpose-built, maintained actions that handle authentication, error handling, and output parsing natively.

**Common replacements to look for:**

| CLI / Tool Usage | Official GHA Replacement | Vendor |
|---|---|---|
| `az login`, `az cli ...` | `Azure/login`, `Azure/cli` | Microsoft |
| `aws sts`, `aws s3`, `aws ecr ...` | `aws-actions/configure-aws-credentials`, `aws-actions/amazon-ecr-login` | AWS |
| `gcloud auth`, `gcloud ...` | `google-github-actions/auth`, `google-github-actions/setup-gcloud` | Google |
| `docker login`, `docker build`, `docker push` | `docker/login-action`, `docker/build-push-action` | Docker |
| `kubectl apply`, `helm upgrade` | `Azure/k8s-deploy`, `azure/k8s-set-context` | Microsoft |
| `terraform init/plan/apply` | `hashicorp/setup-terraform` | HashiCorp |
| `node/npm/yarn` setup | `actions/setup-node` | GitHub |
| `java/maven/gradle` setup | `actions/setup-java` (with `cache: maven` or `cache: gradle`) | GitHub |
| `./gradlew` build/test | `gradle/actions/setup-gradle` + `run: ./gradlew build` | Gradle |
| `mvn spring-boot:build-image` / `./gradlew bootBuildImage` | Native Spring Boot buildpack (no action needed, runs as `run:` step) | Spring |
| `python/pip` setup | `actions/setup-python` | GitHub |
| `go` setup | `actions/setup-go` | GitHub |
| `slack` webhook/API calls | `slackapi/slack-github-action` | Slack |
| `sonar-scanner` | `sonarsource/sonarqube-scan-action` | SonarSource |

Record any CLI-to-action opportunities found. These will be filed as GitHub issues to the repo as recommended action replacements, with each action pinned to a full commit SHA.

## Step 8: Scope Assessment

Evaluate whether this migration request is appropriately sized.

**Too Large (split into multiple migration specs):**

- Organization-wide migration of all Jenkins pipelines at once
- 10+ pipelines migrated simultaneously
- Migrating pipelines while also changing build tools (e.g., Maven to Gradle)
- Complete shared library ecosystem migration in one spec
- Migrating pipelines plus provisioning new infrastructure (runners, cloud accounts)

**Too Small (migrate directly without formal spec):**

- Single Jenkinsfile with 2-3 simple stages and no plugins
- Pipeline that only runs a single shell script
- Jenkinsfile that is already mostly shell commands with no plugin dependencies

**Just Right (perfect for this workflow):**

- 1-3 related pipelines with moderate plugin usage
- Single complex pipeline with shared library dependencies
- Pipeline with significant credential management and deployment logic
- Multibranch pipeline with environment-specific deployment strategies

### Report Scope Assessment To User

- **ALWAYS** inform the user of the result of the scope assessment.
- If the scope appears inappropriate, **ALWAYS** pause the conversation to suggest alternatives and get input from the user.

## Output Requirements

**Format:** Markdown (`.md`)
**Directory:** `./docs/specs/[NN]-migration-[pipeline-name]/`
**Filename:** `[NN]-discovery-[pipeline-name].md`
**Example:** `./docs/specs/01-migration-build-pipeline/01-discovery-build-pipeline.md`

### Discovery Document Structure

```markdown
# [NN] Discovery Report — [Pipeline Name]

## Repository Details
[Default branch name and Git metadata]

## Pipeline Inventory
[Table of all Jenkinsfiles found with classification]

## Matrix Build Opportunities
[Table of matrix candidates with Jenkins pattern, suggested dimensions, and combination counts — or explicit statement that none were found]

## Plugin Dependency Matrix
[Table of all plugins detected with purpose and usage count]

## Credentials Catalog
[Table of all credential references — IDs and types only, NEVER values]

## Agent/Runner Mapping
[Table mapping Jenkins agents to suggested GHA runners]

## Shared Library Catalog
[Table of shared libraries with functions used and migration strategies]

## Integration Points
[Tables of external services, deployment targets, notifications, approval gates]

## Risk Assessment
[Table rating each component: High/Medium/Low risk with rationale]

## Open Questions
[Any questions that could not be answered from the pipeline files alone]
```

## Critical Constraints

**Boundaries** — This is a read-only assessment. Do not modify repository files or begin writing the migration spec. If no Jenkinsfile or pipeline reference has been provided, ask the user before proceeding.

**Secrets safety** — Only document credential IDs and types. Never extract, display, or log actual secret values — this protects the user if the discovery report is committed to the repo.

**Completeness requirements** — Every component discovered must appear in the report. Specifically:
- Every credential reference (by ID only)
- Every pipeline classified by type, complexity, and trigger
- Every agent configuration mapped to a GHA runner strategy
- Every shared library dependency identified
- A risk assessment for each component
- A scope assessment presented to the user before generating the report

If the Jenkinsfile alone is insufficient (e.g., shared library source not available), flag it explicitly rather than guessing.

## What Comes Next

Once this discovery report is complete and reviewed, instruct the user to run `/SDM-2-generate-migration-spec`. The discovery report will serve as the primary input for generating the detailed migration specification.
