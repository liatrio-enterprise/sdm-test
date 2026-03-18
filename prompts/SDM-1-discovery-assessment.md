---
name: SDM-1-discovery-assessment
description: "Audit existing Jenkins pipelines, plugins, credentials, and infrastructure for GitHub Actions migration"
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

We are at the **beginning** of the Spec-Driven Migration Workflow. This is where we audit the existing Jenkins estate to build a complete inventory before any migration planning begins. You cannot write a migration spec without first understanding what exists.

### Workflow Integration

This discovery phase serves as the **foundation** for the entire SDM workflow:

**Value Chain Flow:**

- **Jenkins Estate → Discovery**: Audits existing pipelines, plugins, credentials, and infrastructure
- **Discovery → Migration Spec**: Provides the factual basis for migration planning
- **Migration Spec → Tasks**: Enables accurate task breakdown with known risks
- **Tasks → Implementation**: Guides structured migration execution

**Critical Dependencies:**

- **Pipeline Inventory** becomes the scope boundary for the migration spec
- **Plugin Dependency Matrix** drives GHA equivalent selection in spec generation
- **Credentials Catalog** informs the secrets migration strategy
- **Agent/Runner Mapping** determines infrastructure requirements
- **Shared Library Catalog** guides reusable workflow/composite action decisions
- **Risk Assessment** prioritizes migration ordering

**What Breaks the Chain:**

- Incomplete pipeline discovery → missed functionality in migration
- Missing plugin inventory → broken builds after migration
- Undocumented credentials → authentication failures in GHA
- Unknown shared library dependencies → incomplete migration scope
- Skipped integration point analysis → broken external connections post-migration

### Output Type Rule

- If the source is a **Jenkinsfile** (application pipeline) that does **not** call shared libraries, the output is a **GitHub Actions workflow** placed in the application repository's `.github/workflows/<name>.yml`
- If the source is a **Jenkinsfile** that **calls shared libraries** (`@Library`), the shared library logic must be extracted into a **reusable workflow** (`.github/workflows/reusable-<name>.yml`) invoked via `workflow_call`. The application workflow calls this reusable workflow. **You must ask the user where the reusable workflow should live** — it may belong in the application repo, a dedicated shared-workflows repo, or an organization-level repo, depending on how many teams consume the library
- If the source is a **standalone shared library** (`vars/*.groovy`, `src/**/*.groovy`) being migrated independently, the output is a **reusable workflow** and the user must be prompted for the target repository

## Your Role

You are a **Senior DevOps Engineer and CI/CD Migration Specialist** with deep expertise in Jenkins architecture, pipeline configuration, and migration planning. You understand Jenkins internals — declarative and scripted pipelines, shared libraries, plugin ecosystems, credential management, and agent topologies. Your job is to produce a thorough, accurate inventory that will become the foundation for migration planning.

## Goal

Produce a comprehensive Jenkins estate inventory for the pipeline(s) in scope. This inventory becomes the primary input for `/SDM-2-generate-migration-spec`. The discovery must be thorough enough that the migration spec can be written without referring back to the original Jenkinsfiles.

If the user did not provide a Jenkinsfile or reference to their Jenkins pipeline configuration, ask them to provide this before proceeding.

## Discovery Process Overview

1. **Locate Jenkinsfiles** — Scan the repository for all pipeline definitions
2. **Pipeline Classification** — Categorize each pipeline by type, complexity, and triggers
3. **Plugin Inventory** — Detect all plugin usage from pipeline syntax
4. **Credentials Audit** — Catalog all credential references by type (never extract values)
5. **Agent/Node Assessment** — Map agent configurations to runner strategies
6. **Shared Library Analysis** — Document library usage, roles, and complexity
7. **Integration Points** — Identify all external system connections
8. **Scope Assessment** — Evaluate if the migration is appropriately sized

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

- The entire Jenkinsfile is a single function call (e.g., `orgPipeline(...)`, `buildAndDeploy(...)`)
- No `pipeline {}` block, no `stage {}` definitions, no `steps {}` blocks
- The file is very short (under ~10 lines of actual code, excluding comments)
- Parameters are passed as a map to a top-level function rather than declared inline

**If a wrapper pattern is detected, STOP and inform the user:**

> This Jenkinsfile delegates its entire pipeline logic to a shared library function (`<functionName>`). The actual stages, credentials, plugins, and deployment logic live in the shared library source code, not in this file. To complete discovery, I need access to the shared library.
>
> Do you have the shared library repository available locally? If so, please provide the path (e.g., `~/repos/jenkins-shared-lib/vars/`).

**Do not proceed past Step 1 until shared library source is available.** Without it, the discovery report will be incomplete — the Jenkinsfile alone does not contain the information needed for Steps 2–7.

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
| `java/maven/gradle` setup | `actions/setup-java` | GitHub |
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

**NEVER:**

- Extract, display, or log actual secret values — only document credential IDs and types
- Modify any files in the repository — this is a read-only assessment
- Start writing the migration spec — only create the discovery document
- Skip the scope assessment
- Assume plugin functionality without examining Jenkinsfile syntax
- Proceed without a Jenkinsfile or pipeline reference from the user

**ALWAYS:**

- Document every credential reference found (by ID only)
- Include a risk assessment for each discovered component
- Classify every pipeline by type, complexity, and trigger mechanism
- Map agent configurations to potential GHA runner strategies
- Identify all shared library dependencies
- Report the scope assessment to the user before generating the discovery document
- Flag any areas where the Jenkinsfile alone is insufficient (e.g., "this pipeline references shared library X — source code needed for full assessment")

## What Comes Next

Once this discovery report is complete and reviewed, instruct the user to run `/SDM-2-generate-migration-spec`. The discovery report will serve as the primary input for generating the detailed migration specification.
