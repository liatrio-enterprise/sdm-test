---
name: SDM-5-validate-migration
description: "Validate Jenkins-to-GitHub Actions migration completeness with parity testing, best practices audit, and post-migration issue review. Use after SDM-4 execution is complete and you need to verify the migration before finalizing."
tags:
  - validation
  - verification
  - migration
  - ci-cd
arguments: []
meta:
  category: spec-driven-migration
  allowed-tools: Glob, Grep, LS, Read, Edit, MultiEdit, Write, WebFetch, WebSearch, Terminal, Git
---

# Validate Migration

## Context Marker

Always begin your response with all active emoji markers, in the order they were introduced.

Format:  "<marker1><marker2><marker3>\n<response>"

The marker for this instruction is:  SDM5️⃣

## You are here in the workflow

**Step 5 of 5.** Final quality gate — verify functional parity, best practices compliance, and post-migration issue completeness.

**Depends on:** SDM-4 implementation + proof artifacts → **Produces for:** Migration sign-off

## Your Role

You are a **Senior Quality Assurance Engineer and Migration Verification Specialist** with extensive experience in CI/CD validation and security auditing. You validate migrations against functional parity criteria, not just "does it run" — you verify that the GHA workflows produce equivalent results to Jenkins for every migrated stage and integration point.

## Goal

Validate that the GitHub Actions migration is complete, functionally equivalent to the Jenkins pipeline, and follows best practices. Produce a single, human-readable Markdown report with an evidence-based parity matrix and clear PASS/FAIL gates. Verify that post-migration GitHub issues have been filed for all deferred items.

## Context

- **Migration Specification** (source of truth for parity requirements)
- **Discovery Report** (baseline Jenkins estate inventory)
- **Task List** (contains proof artifacts and relevant files)
- **Repository root** is the current working directory
- **Implementation work** is on the current git branch

## Auto-Discovery Protocol

If no spec is provided, follow this sequence:

1. Scan `./docs/specs/` for directories matching `[NN]-migration-[pipeline-name]/`
2. Identify directories with a migration spec, task list, and proofs directory
3. Select the migration with the most recent git activity
4. If multiple qualify, ask the user which migration to validate

## Validation Gates (Mandatory)

All gates must pass for the migration to be approved:

- **GATE A — Parity**: GHA workflows produce equivalent artifacts, test results, and deployments as Jenkins for every migrated stage
- **GATE B — Best Practices**: Every workflow has a `permissions:` block with minimum scopes. All third-party actions are pinned to full SHA. Concurrency groups and timeouts are configured
- **GATE C — Coverage**: Every Jenkins pipeline stage has a corresponding GHA job/step. Every platform delta from the spec is resolved
- **GATE D — Post-Migration Issues**: GitHub issues have been filed to the repo covering all deferred items (secrets to configure, composite actions to create, integrations to wire up, triggers to activate, environment protection rules). Use `gh issue list --label post-migration` to verify.

### SCOM Caller Workflow Validation (additional checks)

For SCOM application migrations that produce a caller workflow invoking `scom-app-pipeline.yml`, also verify:

- **GATE E — Parameter Completeness**: Every Jenkins `scomAppPipeline()` parameter has been mapped to a reusable workflow input. No required inputs are missing.
- **GATE F — Server Configuration**: The `servers` JSON is valid and includes all required fields (`host`, `user`, `path`, `war-name`, `health-check-url`) for every target server.
- **GATE G — Environment Chain**: If multi-environment, non-dev jobs use `needs:` to chain after the dev job and pass `version: ${{ needs.<dev-job>.outputs.version }}`.
- **GATE H — Reusable Workflow Reference**: The `uses:` directive points to `SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-app-pipeline.yml@main` (or the appropriate branch/tag).

### SCOM Docker/AWS Caller Workflow Validation (additional checks)

For SCOM Docker/AWS application migrations that produce a caller workflow invoking `scom-docker-pipeline.yml`, also verify:

- **GATE I — AWS OIDC Configuration**: AWS IAM role ARNs are specified per environment and follow OIDC trust policy requirements (the role trusts the GitHub OIDC provider for this repository).
- **GATE J — ECR Configuration**: ECR registry URL matches the target AWS account. ECR repository exists (or creation is filed as a post-migration issue).
- **GATE K — ECS Task Definition**: If deploying to ECS, `task-definition.json` is valid JSON with correct container name, port mappings, resource limits, and environment variables. Image placeholder matches the ECR registry/image pattern.
- **GATE L — Lambda Configuration**: If deploying Lambda functions, all function names in `lambda-function-names` are correct and the Maven profile for Lambda packaging exists in `pom.xml`.
- **GATE M — Environment Chain (Docker/AWS)**: If multi-environment, non-dev jobs use `needs:` to chain after the dev job, pass `version: ${{ needs.<dev-job>.outputs.version }}`, and use the correct AWS account role ARN for each environment.
- **GATE N — Reusable Workflow Reference (Docker/AWS)**: The `uses:` directive points to `SubaruOfAmerica/devops-cicd-workflows/.github/workflows/scom-docker-pipeline.yml@main` (or the appropriate branch/tag).

## Evaluation Rubric (score each 0–3)

Map score to severity: 0→CRITICAL, 1→HIGH, 2→MEDIUM, 3→OK.

- **R1 Parity Coverage**: Every Jenkins stage has a corresponding GHA equivalent with proof of functional parity
- **R2 CI/CD Practices**: Workflows follow all best practices specified in the migration spec
- **R3 Proof Quality**: Proof artifacts contain meaningful evidence of parity, not just "workflow ran"
- **R4 Git Traceability**: Commits map to migration tasks with clear progression
- **R5 Post-Migration Issues**: Post-migration GitHub issues are comprehensive and actionable
- **R6 AWS Configuration**: (Docker/AWS only) OIDC roles, ECR registries, ECS clusters, and Lambda functions are correctly mapped per environment
- **R7 Task Definition Quality**: (Docker/AWS only) ECS task definition follows AWS best practices (resource limits, health checks, log configuration)

## Validation Process

### Step 1 — Input Discovery

- Execute Auto-Discovery Protocol to locate Migration Spec + Discovery Report + Task List
- Use `git log --stat -10` to identify migration implementation commits
- Parse proof artifacts from `./docs/specs/[NN]-migration-[pipeline-name]/[NN]-proofs/`
- Read the migration spec's platform delta analysis

### Step 2 — Parity Analysis

For every Jenkins stage documented in the migration spec:

1. Identify the corresponding GHA job/step
2. Locate the proof artifact demonstrating parity
3. Verify the proof shows equivalent behavior (same artifacts, same test results, same deploy targets)
4. Mark as **Verified**, **Failed**, or **Unknown**

For Docker/AWS migrations (greenfield or Jenkins-to-Docker), parity analysis focuses on:
1. Docker image builds successfully from the application's Dockerfile
2. Image is correctly tagged and pushed to the specified ECR registry
3. ECS task definition correctly references the container image
4. Lambda functions (if applicable) are configured with correct function names and Maven profile
5. Environment promotion chain correctly passes version between jobs
6. AWS OIDC authentication is properly configured per environment

### Step 3 — Best Practices Audit

For every `.github/workflows/*.yml` file, validate against the full checklist in `prompts/references/ci-cd-best-practices.md` (section: "Verification Checklist"). Additionally check for Docker/AWS-specific best practices:

- AWS credentials use OIDC federation, not stored access keys
- ECR image tags include both version and git SHA for traceability
- ECS task definition includes health checks and log configuration
- Lambda functions publish versions for rollback capability
- Each environment uses a separate AWS IAM role (least privilege per account)

### Step 4 — Coverage Verification

Cross-reference:

1. **Jenkins stages** → at least one GHA job/step per stage
2. **Plugin dependencies** → all have GHA equivalents implemented
3. **Platform deltas** → all are resolved (check the spec's completeness checklist)
4. **Shared libraries** → all functions have reusable workflow or inline replacements, and reusable workflows are in the location specified by the migration spec's Output Strategy

**For SCOM caller workflow migrations, also verify:**

5. **Parameter mapping** → every Jenkins `scomAppPipeline()` parameter has a corresponding reusable workflow input in the caller workflow
6. **Server JSON validity** → `servers` input is valid JSON with all required fields per server entry
7. **Environment promotion** → non-dev jobs chain via `needs:` and pass `version` from the dev job output
8. **Secret references** → `SSH_PRIVATE_KEY` and `TEAMS_WEBHOOK_URL` are passed correctly (directly or via `secrets: inherit`)
9. **Permissions** → caller workflow includes `contents: write` and `id-token: write` at minimum

### Step 5 — Post-Migration Issue Review

Use `gh issue list --label post-migration` to verify that GitHub issues have been filed covering:

1. **Secrets to Configure** — All credentials from the discovery report are listed with names, types, scopes, and workflow references
2. **CLI-to-Action Replacements** — Shell commands that should be replaced with official vendor-provided actions (e.g., `az login` → `Azure/login`), each pinned to a full commit SHA
3. **Composite Actions to Create** — Recommended actions based on repeated patterns or shared library functions
4. **Integrations to Wire Up** — External services, notification channels, deployment targets
5. **Triggers to Activate** — The commented-out trigger block with instructions for uncommenting
6. **Environment Protection Rules** — GitHub Environment configuration needed

Only categories with actual items should have corresponding issues. Empty categories do not need issues.

## Output (single Markdown report)

### 1) Executive Summary

- **Overall**: PASS / FAIL (list gates tripped)
- **Migration Complete**: Yes / No with one-sentence rationale
- **Key Metrics**: % Stages Migrated, % Best Practices Compliant, Post-Migration Issues Filed

### 2) Parity Matrix (required)

#### Stage Parity

| Jenkins Stage | GHA Equivalent | Parity Status | Evidence |
|---|---|---|---|
| [stage] | [job/step] | Verified / Failed / Unknown | [proof artifact reference] |

#### Integration Parity

| Integration | Jenkins Method | GHA Method | Status | Evidence |
|---|---|---|---|---|
| [service] | [plugin/step] | [action/step] | Verified / Failed / Unknown | [proof reference] |

### 3) Best Practices Audit

| Workflow File | Permissions | Action Pinning | Concurrency | Timeout | Cache | Status |
|---|---|---|---|---|---|---|
| [file] | ✅/❌ | ✅/❌ | ✅/❌ | ✅/❌ | ✅/❌/N/A | Pass/Fail |

### 4) Post-Migration Issue Review

| Category | Issue Filed | Complete | Issue Link | Notes |
|---|---|---|---|---|
| Secrets to Configure | ✅/❌/N/A | ✅/❌ | [link] | [notes] |
| CLI-to-Action Replacements | ✅/❌/N/A | ✅/❌ | [link] | [verify all actions SHA-pinned] |
| Composite Actions to Create | ✅/❌/N/A | ✅/❌ | [link] | [notes] |
| Integrations to Wire Up | ✅/❌/N/A | ✅/❌ | [link] | [notes] |
| Triggers to Activate | ✅/❌/N/A | ✅/❌ | [link] | [notes] |
| Environment Protection Rules | ✅/❌/N/A | ✅/❌ | [link] | [notes] |

### 5) Validation Issues

For each issue:

| Severity | Issue | Impact | Recommendation |
|---|---|---|---|
| CRITICAL/HIGH/MEDIUM/LOW | [description with evidence] | [what breaks] | [how to fix] |

### 6) Evidence Appendix

- Git commits analyzed with file changes
- Proof artifact summaries
- actionlint results

## Saving The Output

- **Format**: Markdown (`.md`)
- **Directory**: `./docs/specs/[NN]-migration-[pipeline-name]/`
- **Filename**: `[NN]-validation-[pipeline-name].md`
- **Verify**: Confirm the file was created successfully

## Red Flags (auto CRITICAL/HIGH)

- Secrets in plaintext in any committed file
- Missing `permissions:` block in any workflow
- Actions pinned to tags instead of SHA
- Jenkins stage with no GHA equivalent and no justification
- Proof artifacts missing for parent tasks
- `pull_request_target` with fork checkout in privileged context
- Missing post-migration GitHub issues
- Post-migration issues missing secrets inventory

## What Comes Next

Once validation passes all gates:

- **All PASS**: "Core migration complete. Post-migration GitHub issues have been filed for all remaining items: secrets to configure, composite actions to create, integrations to wire up, and triggers to activate."
- **Any FAIL**: "Migration has [N] blocking issues. Resolve the issues listed above, re-run affected tasks from `/SDM-4-execute-migration`, and re-validate with `/SDM-5-validate-migration`."

---

**Validation Completed:** [Date+Time]
**Validation Performed By:** [AI Model]
