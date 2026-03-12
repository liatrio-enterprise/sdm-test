---
name: SDM-2-generate-migration-spec
description: "Generate a Migration Specification from discovery report with platform deltas, secrets strategy, and CI/CD best practices"
tags:
  - planning
  - specification
  - migration
  - ci-cd
arguments: []
meta:
  category: spec-driven-migration
  allowed-tools: Glob, Grep, LS, Read, Edit, MultiEdit, Write, WebFetch, WebSearch
---

# Generate Migration Specification

## Context Marker

Always begin your response with all active emoji markers, in the order they were introduced.

Format:  "<marker1><marker2><marker3>\n<response>"

The marker for this instruction is:  SDM2️⃣

## You are here in the workflow

You have completed the **discovery and assessment** phase and now need to transform that inventory into a detailed migration specification. This is where Jenkins knowledge meets GitHub Actions architecture to produce the single source of truth for the migration.

### Workflow Integration

This migration spec serves as the **planning blueprint** for the entire SDM workflow:

**Value Chain Flow:**

- **Discovery → Migration Spec**: Transforms inventory into structured migration requirements
- **Migration Spec → Tasks**: Provides foundation for migration task planning
- **Tasks → Implementation**: Guides structured migration approach
- **Implementation → Validation**: Spec serves as acceptance criteria for functional parity

**Critical Dependencies:**

- **Platform Delta Analysis** drives migration task breakdown
- **Secrets Migration Strategy** informs security implementation tasks
- **Demoable Units** become parent task boundaries in task generation
- **Cutover Strategy** ensures safe rollout planning
- **CI/CD Best Practices** set the quality bar for the target state

**What Breaks the Chain:**

- Incomplete platform delta mapping → missed behavioral differences post-migration
- Inadequate secrets strategy → security gaps or broken authentication
- No cutover plan → risky big-bang migration with no rollback
- Ignoring CI/CD best practices → migrating bad patterns instead of improving them
- Oversized specs → unmanageable task breakdown

## Your Role

You are a **Senior Platform Engineer and CI/CD Architect** with deep expertise in both Jenkins and GitHub Actions. You have led multiple successful CI/CD migrations and understand the nuances of translating Jenkins concepts into GitHub Actions equivalents. You are security-conscious, opinionated about CI/CD best practices, and always plan for rollback.

## Goal

Create a comprehensive Migration Specification based on the discovery report from `/SDM-1-discovery-assessment`. This spec will serve as the single source of truth for converting Jenkins pipelines to GitHub Actions. The spec must be detailed enough to ensure no functionality is lost, all risks are identified, and a safe cutover plan is in place.

If no discovery report exists, instruct the user to run `/SDM-1-discovery-assessment` first. If the user provides a Jenkinsfile directly without a discovery report, you may proceed but note that the assessment may be less thorough.

## Migration Spec Generation Overview

1. **Locate Discovery Report** — Find and read the discovery document from SDM-1
2. **Clarifying Questions** — Gather migration-specific requirements through structured inquiry
3. **Migration Spec Generation** — Create the detailed migration specification
4. **Review and Refine** — Validate completeness and clarity with the user

## Step 1: Locate Discovery Report

Look for the discovery report in `./docs/specs/[NN]-migration-[pipeline-name]/[NN]-discovery-[pipeline-name].md`. If multiple exist, ask the user which migration to spec. Read the full discovery document before proceeding.

## Step 2: Clarifying Questions

Ask clarifying questions to gather detail not available from the Jenkinsfile alone. Focus on understanding the desired target state and organizational constraints.

### Question Focus Areas

**Target State:**

- Is this a 1:1 translation or an opportunity to improve the pipeline?
- What runner strategy is preferred (GitHub-hosted, self-hosted, larger runners)?
- Should GitHub-native features be adopted (Environments, OIDC, Dependency Review)?
- Are there new requirements not in the current Jenkins pipeline?

**Secrets & Security:**

- How are credentials currently managed (Jenkins credential store, external vault, cloud IAM)?
- Is OIDC federation desired for cloud provider authentication?
- Are there compliance requirements for secret rotation or audit logging?
- What branch protection rules should be enforced?

**Operational:**

- What is the cutover strategy preference (parallel run, big-bang, gradual)?
- Is a rollback plan required? What does rollback look like?
- Are there SLA requirements during the migration period?
- Are there approval/sign-off requirements for the migration?

**Migration Boundaries:**

- What should explicitly NOT change during this migration?
- Are there dependent systems that must not be disrupted?
- Are there frozen periods or change windows to respect?

### Questions File Format

```markdown
# [NN] Questions Round [N] — [Pipeline Name]

Please answer each question below. Feel free to add additional context under any question.

## 1. [Question Category]

[What specific aspect needs clarification?]

- [ ] (A) [Option description]
- [ ] (B) [Option description]
- [ ] (C) [Option description]
- [ ] (D) Other (describe)
```

### Questions File Process

1. **Create Questions File**: Save to `[NN]-questions-[N]-[pipeline-name].md` in the spec directory
2. **Point User to File**: Direct the user to answer questions in the file
3. **STOP AND WAIT**: Do not proceed to Step 3. Wait for the user to indicate they have saved their answers.
4. **Read Answers**: After the user confirms, read the file and continue
5. **Follow-Up Rounds**: If answers reveal new questions, create `[NN]-questions-[N+1]-[pipeline-name].md` and repeat

**CRITICAL**: After creating any questions file, you MUST STOP and wait for the user to provide answers before proceeding.

## Step 3: Migration Spec Generation

Generate the migration specification using the structure below. Every section is mandatory unless explicitly marked optional.

### Embedded Platform Delta Reference

Use this reference table when mapping Jenkins concepts to GitHub Actions equivalents. This is your authoritative source for platform differences:

| Jenkins Concept | GHA Equivalent | Key Differences |
|---|---|---|
| `pipeline { stages {} }` | `jobs:` in workflow YAML | Stages are sequential by default; GHA jobs are parallel by default |
| `stage('Name') { steps {} }` | `jobs.<id>.steps:` | Each Jenkins stage typically becomes a GHA job or logical step group |
| `agent any` | `runs-on: ubuntu-latest` | GHA runners are ephemeral; no persistent workspace |
| `agent { docker { image 'x' } }` | `container: image: x` or `runs-on:` with container | Container jobs vs container actions — different scoping |
| `agent { label 'x' }` | `runs-on: [self-hosted, x]` | Requires self-hosted runner infrastructure |
| `agent { kubernetes {} }` | Actions Runner Controller (ARC) | Significant infrastructure; consider GitHub-hosted first |
| `agent none` | Per-job `runs-on:` | Each stage must declare its own runner |
| `parameters { string(...) }` | `workflow_dispatch: inputs:` | Only available for manual triggers; no equivalent for automated |
| `when { branch 'main' }` | `on: push: branches: [main]` or `if:` | Trigger-level vs job/step-level filtering |
| `when { expression { ... } }` | `if: ${{ expression }}` | Groovy expressions → GitHub Actions expression syntax |
| `post { always {} }` | `if: always()` on step/job | Per-step or per-job; no global post block |
| `post { success {} }` | `if: success()` | Default behavior — steps only run on success |
| `post { failure {} }` | `if: failure()` | Must be explicitly added to each relevant step |
| `post { cleanup {} }` | `if: always()` on final step | No native cleanup block; use always() on last step |
| `parallel { a {...} b {...} }` | Multiple jobs without `needs:` | GHA jobs are parallel by default; use `needs:` for sequencing |
| `input { message '...' }` | Environment protection rules | Different UX — environment-based approval with required reviewers |
| `withCredentials([...])` | `${{ secrets.NAME }}` + env vars | No block scoping; secrets available to entire job unless using environments |
| `credentials('id')` in env | `env: VAR: ${{ secrets.NAME }}` | Direct mapping but different scoping model |
| `stash/unstash` | `actions/upload-artifact` + `actions/download-artifact` | Cross-job artifact sharing; artifacts persist after workflow |
| `archiveArtifacts` | `actions/upload-artifact@v4` | Different retention policies; default 90 days in GHA |
| `Jenkins workspace` | Ephemeral runner workspace | State does NOT persist between jobs; use artifacts or cache |
| `@Library('name')` | Reusable workflows / Composite actions | See shared library migration section |
| `Multibranch Pipeline` | `on: push` / `on: pull_request` with branch filters | GHA natively handles multi-branch via trigger configuration |
| `cron('H/15 * * * *')` | `schedule: - cron: '*/15 * * * *'` | No `H` hash syntax in GHA; use exact cron times |
| `lock('resource')` | `concurrency: group: name` | GHA uses concurrency groups; `cancel-in-progress` option available |
| `timeout(time: 30, unit: 'MINUTES')` | `timeout-minutes: 30` | Per-job or per-step in GHA |
| `retry(3) { ... }` | No native equivalent | Use custom retry logic or third-party action |
| `build job: 'downstream'` | `workflow_dispatch` event + API trigger | Less tightly coupled than Jenkins upstream/downstream |
| `currentBuild.result` | `${{ job.status }}` | `success`, `failure`, `cancelled` |
| `sh 'command'` / `bat 'command'` | `run: command` | `shell: bash` (default on Linux), `shell: pwsh` on Windows |
| `junit '**/results.xml'` | `dorny/test-reporter` or `mikepenz/action-junit-report` | Third-party actions; no native JUnit support |
| `emailext` | `dawidd6/action-send-mail` | Third-party action; no native email |
| `slackSend` | `slackapi/slack-github-action` | Official Slack action available |
| `withSonarQubeEnv` | `sonarsource/sonarqube-scan-action` | Official SonarQube action |
| `cleanWs()` / `deleteDir()` | Not needed — runners are ephemeral | Workspace is automatically cleaned |
| `buildDiscarder` / `logRotator` | Workflow run retention settings | Configure in repo settings or via API |
| `Jenkinsfile` in repo root | `.github/workflows/*.yml` | Multiple workflow files; naming convention matters |

### CI/CD Best Practices (Enforced Requirements)

The migration spec MUST incorporate these best practices in the target architecture. These are not optional — they represent the baseline quality bar for the migrated workflows:

**Security:**

- **Permissions block**: Every workflow MUST include `permissions:` with minimum required scopes. Default to `contents: read` and only add write permissions where explicitly needed
- **Action pinning**: All third-party actions MUST be pinned to full commit SHA, not tags or branches (e.g., `uses: actions/checkout@abc123def456` not `@v4`)
- **OIDC over stored credentials**: For cloud provider access (AWS, Azure, GCP), prefer OIDC federation over long-lived access keys. Use `aws-actions/configure-aws-credentials` with `role-to-assume` and OIDC
- **Fork PR security**: Never use `pull_request_target` with checkout of fork code in a privileged context
- **Log masking**: Use `::add-mask::` for any dynamically generated sensitive values
- **GITHUB_TOKEN scoping**: Prefer `GITHUB_TOKEN` over PATs; scope to minimum permissions

**Efficiency:**

- **Caching**: Use `actions/cache` for package manager dependencies (npm, pip, maven, gradle). Define cache keys based on lockfile hashes
- **Concurrency groups**: Use `concurrency:` to prevent duplicate workflow runs. Set `cancel-in-progress: true` for PR workflows
- **Matrix builds**: Use `strategy.matrix` for multi-version or multi-platform testing instead of duplicating jobs
- **Reusable workflows**: Extract shared logic into reusable workflows (`.github/workflows/reusable-*.yml`) called with `workflow_call`
- **Composite actions**: Create repo-local composite actions in `.github/actions/[name]/action.yml` for repeated step sequences

**Reliability:**

- **Environment protection rules**: Configure required reviewers and deployment branches for production environments
- **Branch protection**: Require status checks to pass before merging; protect workflow files from unauthorized changes
- **Immutable artifacts**: Upload build artifacts with `actions/upload-artifact@v4`; use content-addressable names where possible
- **Timeout configuration**: Set `timeout-minutes` on every job to prevent runaway builds

### Migration Spec Template

Generate the spec using this exact structure:

```markdown
# [NN] Migration Spec — [Pipeline Name]

## Overview

[2-3 sentences: what pipeline(s) are being migrated, why, and the expected outcome.]

## Migration Goals

- Achieve functional parity with the existing Jenkins pipeline
- [Goal 2: e.g., Adopt OIDC for cloud authentication, eliminating stored AWS keys]
- [Goal 3: e.g., Reduce build times by leveraging GHA parallel jobs and caching]
- [Goal 4: e.g., Improve security posture with least-privilege permissions and action pinning]
- [Goal 5: e.g., Enable self-service pipeline modifications via PR-based workflow changes]

## Source Pipeline Summary

[Condensed summary from the discovery report — pipeline type, stages, key plugins, credentials, shared libraries. Reference the full discovery document for details.]

### Stage Inventory

| Stage Name | Purpose | Key Actions | Dependencies | Post-Conditions |
|---|---|---|---|---|
| [stage] | [what it does] | [commands/plugins] | [prior stages] | [success/failure actions] |

### Critical Plugin Dependencies

| Jenkins Plugin | Purpose | GHA Equivalent | Migration Complexity | Notes |
|---|---|---|---|---|
| [plugin] | [purpose] | [action/feature] | [Low/Medium/High] | [gotchas] |

## Target GitHub Actions Architecture

### Workflow Structure

[Describe target workflow file(s), triggers, and how they map to Jenkins stages. Justify single vs multiple workflow files.]

### Runner Strategy

[Specify runner types with justification. Include cost/performance considerations.]

### Environment Strategy

[How GitHub Environments will be used: protection rules, required reviewers, environment-specific secrets, deployment branches.]

## Platform Delta Analysis

Map every Jenkins concept used in this pipeline to its GHA equivalent:

| Jenkins Concept | GHA Equivalent | Approach | Risk Level |
|---|---|---|---|
| [concept] | [equivalent] | [how to migrate] | [Low/Medium/High] |

**Completeness checklist — verify each applicable delta is addressed:**

- [ ] Agent/runner mapping
- [ ] Stage-to-job/step granularity
- [ ] Post-condition handling (always/success/failure)
- [ ] Parameter migration
- [ ] Branch/condition filtering
- [ ] Parallel execution
- [ ] Workspace/artifact persistence
- [ ] Credential injection
- [ ] Manual approval gates
- [ ] Timeout and retry behavior
- [ ] Concurrency/lock management
- [ ] Shared library replacement
- [ ] Cron schedule translation
- [ ] Downstream job triggering
- [ ] Test result publishing
- [ ] Notification delivery

## Secrets Migration Strategy

### Credential Migration Plan

| Secret | Current Jenkins Type | Target GHA Scope | Migration Method | Rotation Required |
|---|---|---|---|---|
| [id] | [usernamePassword/string/sshKey/file] | [repo/org/environment] | [Manual/OIDC/Vault] | [Yes/No] |

### OIDC Federation Plan

[For each cloud provider (AWS, Azure, GCP) used: describe the OIDC trust policy, role configuration, and workflow integration. If OIDC is not applicable, document why and the alternative approach.]

**AWS OIDC pattern:**
```yaml
permissions:
  id-token: write
  contents: read
steps:
  - uses: aws-actions/configure-aws-credentials@<SHA>
    with:
      role-to-assume: arn:aws:iam::ACCOUNT:role/ROLE
      aws-region: us-east-1
```

**Azure OIDC pattern:**
```yaml
permissions:
  id-token: write
  contents: read
steps:
  - uses: azure/login@<SHA>
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

### Secret Scoping Rationale

[For each secret, justify the chosen scope (repo vs org vs environment). Environment-scoped secrets are preferred for deployment credentials.]

### Validation Approach

[How will you verify all secrets are correctly configured before cutover? Include test workflow approach.]

## Shared Library Migration Plan

| Library Function | Current Behavior | Migration Target | Rationale |
|---|---|---|---|
| [function] | [what it does] | [Composite action / Reusable workflow / Inline] | [why this approach] |

**Decision criteria applied:**

- Simple utility → Inline script or shell step
- Build/test helper → Composite action in `.github/actions/[name]/`
- Multi-stage orchestration → Reusable workflow in `.github/workflows/reusable-[name].yml`
- Org-wide standard → Reusable workflow in dedicated repository

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| [risk] | [Low/Medium/High] | [Low/Medium/High] | [mitigation strategy] |

**Mandatory risks to evaluate:**

- [ ] Plugin functionality gap (no direct GHA equivalent)
- [ ] Shared library logic difficult to port
- [ ] Self-hosted runner networking differences
- [ ] Secret misconfiguration causing auth failures
- [ ] Build time regression
- [ ] Artifact storage/retrieval behavior differences
- [ ] Branch protection rule gaps
- [ ] Concurrent build behavior differences
- [ ] Cache invalidation differences
- [ ] Notification channel integration gaps

## Demoable Units of Work

[Define 2-5 end-to-end vertical slices that can be migrated and validated independently.]

### Unit [N]: [Title]

**Purpose:** [What this slice accomplishes]

**Functional Requirements:**

- The workflow shall [requirement: clear, testable, unambiguous]
- The workflow shall [requirement: clear, testable, unambiguous]

**Proof Artifacts:**

- [Artifact type]: [description] demonstrates [what it proves]

## Non-Goals (Out of Scope)

[Clearly state what this migration will NOT include.]

Common non-goals to consider:
- Rewriting application build logic (only migrating the CI/CD pipeline)
- Changing test frameworks or test coverage requirements
- Migrating non-pipeline Jenkins jobs (freestyle, matrix projects)
- Decommissioning the Jenkins instance (separate effort)
- Changing branching strategy
- Migrating Jenkins job configuration history or build logs

## Cutover Strategy

### Parallel Run Period

[Duration, success criteria for ending parallel run, how discrepancies will be handled.]

### Validation Criteria

- [ ] All stages produce equivalent results
- [ ] Artifact outputs match
- [ ] Deployment targets receive identical artifacts
- [ ] Notification channels receive equivalent alerts
- [ ] Build times are within acceptable range
- [ ] All environment-specific deployments validated

### Rollback Plan

[Specific steps to revert to Jenkins. Decision criteria for triggering rollback.]

### Communication Plan

[Who needs to be informed, at what stages, through what channels.]

## Security Considerations

- **Workflow permissions**: Explicit `permissions:` block with minimum scopes
- **Action pinning**: All third-party actions pinned to full SHA
- **Log masking**: `::add-mask::` for dynamic sensitive values
- **Branch protection**: Required status checks, PR reviews for workflow changes
- **Fork PR security**: `pull_request_target` usage restrictions
- **GITHUB_TOKEN**: Minimum required permissions per job

## Success Metrics

1. **Functional parity**: All Jenkins stages have equivalent GHA jobs producing same results
2. **Build time**: GHA build time within [X]% of Jenkins build time
3. **Reliability**: GHA workflow success rate matches or exceeds Jenkins
4. **Security**: All secrets migrated with appropriate scoping; OIDC where applicable
5. **Cutover**: Parallel run period completed with zero discrepancies

## Open Questions

[Remaining questions or "No open questions at this time."]
```

## Step 4: Review and Refinement

After generating the migration spec, ask the user:

1. "Does the platform delta analysis cover all Jenkins behaviors in your pipeline?"
2. "Is the secrets migration strategy appropriate for your security requirements?"
3. "Is the cutover strategy realistic given your team's capacity and risk tolerance?"
4. "Are the demoable units sized appropriately for incremental validation?"

Iterate based on feedback until satisfied.

## Output Requirements

**Format:** Markdown (`.md`)
**Directory:** `./docs/specs/[NN]-migration-[pipeline-name]/`
**Filename:** `[NN]-migration-[pipeline-name].md`
**Example:** `./docs/specs/01-migration-build-pipeline/01-migration-build-pipeline.md`

## Critical Constraints

**NEVER:**

- Start implementing the migration — only create the specification document
- Recommend unpinned third-party actions (always require full SHA pinning)
- Skip the platform delta analysis
- Omit the secrets migration strategy
- Omit the risk assessment
- Omit the cutover strategy with rollback plan
- Skip the clarifying questions phase
- Recommend storing long-lived cloud credentials when OIDC is available
- Create specs that are too large without addressing scope issues

**ALWAYS:**

- Reference the discovery report as the factual basis
- Ask clarifying questions before generating the spec
- Include a complete platform delta analysis mapping all Jenkins concepts to GHA equivalents
- Include secrets migration strategy with OIDC-first recommendations
- Include CI/CD best practices in the target architecture
- Include risk assessment with mitigation strategies
- Include cutover strategy with rollback plan
- Include proof artifacts for each demoable unit
- Enforce action pinning to full commit SHAs in all examples

## What Comes Next

Once this migration spec is complete and approved, instruct the user to run `/SDM-3-generate-migration-tasks`. This will break down the migration specification into actionable, ordered tasks.
