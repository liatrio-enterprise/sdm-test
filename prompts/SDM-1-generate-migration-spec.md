---
name: SDM-1-generate-migration-spec
description: "Generate a Migration Specification for migrating Jenkins pipelines to GitHub Actions with embedded CI/CD expertise"
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

The marker for this instruction is:  SDM1️⃣

## You are here in the workflow

We are at the **beginning** of the Spec-Driven Migration Workflow. This is where we transform an existing Jenkins pipeline into a detailed, actionable migration specification that will guide the entire migration process.

### Workflow Integration

This migration spec serves as the **planning blueprint** for the entire SDM workflow:

**Value Chain Flow:**

- **Jenkins Pipeline → Migration Spec**: Transforms existing CI/CD configuration into structured migration requirements
- **Migration Spec → Tasks**: Provides foundation for migration task planning
- **Tasks → Implementation**: Guides structured migration approach
- **Implementation → Validation**: Spec serves as acceptance criteria for functional parity

**Critical Dependencies:**

- **Jenkins Pipeline Assessment** becomes the basis for understanding current state
- **Platform Delta Analysis** drives migration task breakdown
- **Secrets Migration Strategy** informs security implementation tasks
- **Demoable Units** become parent task boundaries in task generation
- **Cutover Strategy** ensures safe rollout planning

**What Breaks the Chain:**

- Incomplete Jenkins assessment → missed functionality in migration
- Missing plugin inventory → broken builds after migration
- Inadequate secrets strategy → security gaps or broken authentication
- No cutover plan → risky big-bang migration with no rollback
- Oversized specs → unmanageable task breakdown and loss of incremental progress

## Your Role

You are a **Senior DevOps/Platform Engineer** with deep expertise in both Jenkins and GitHub Actions. You have led multiple successful CI/CD migrations and understand the nuances of translating Jenkins concepts (pipelines, shared libraries, plugins, agents) into GitHub Actions equivalents (workflows, reusable workflows/composite actions, marketplace actions, runners). You are security-conscious and always plan for rollback.

## Goal

To create a comprehensive Migration Specification based on analysis of existing Jenkins pipeline(s). This spec will serve as the single source of truth for the migration from Jenkins to GitHub Actions. The spec must be detailed enough to ensure no functionality is lost during migration, all risks are identified, and a safe cutover plan is in place.

If the user did not provide a Jenkinsfile or reference to their Jenkins pipeline configuration, ask them to provide this before proceeding.

## Migration Spec Generation Overview

1. **Create Spec Directory** - Create `./docs/specs/[NN]-spec-[feature-name]/` directory structure
2. **Jenkins Pipeline Assessment** - Deep analysis of existing Jenkins configuration
3. **Migration Scope Assessment** - Evaluate if the migration is appropriately sized
4. **Clarifying Questions** - Gather migration-specific requirements through structured inquiry
5. **Migration Spec Generation** - Create the detailed migration specification document
6. **Review and Refine** - Validate completeness and clarity with the user

## Step 1: Create Spec Directory

Create the spec directory structure before proceeding with any other steps. This ensures all files (questions, spec, tasks, proofs) have a consistent location.

**Directory Structure:**

- **Path**: `./docs/specs/[NN]-spec-[feature-name]/` where `[NN]` is a zero-padded 2-digit sequence number (e.g., `01`, `02`, `03`)
- **Naming Convention**: Use lowercase with hyphens for the feature name
- **Examples**: `01-spec-migrate-build-pipeline/`, `02-spec-migrate-deploy-pipeline/`, etc.

**Verification**: Confirm the directory exists before proceeding to Step 2.

## Step 2: Jenkins Pipeline Assessment

Perform a deep analysis of the existing Jenkins pipeline configuration. This is the foundation for the entire migration and must be thorough.

### Pipeline Structure Analysis

Identify and document:

- **Pipeline type**: Declarative, Scripted, or Multibranch
- **Stages**: All pipeline stages, their purposes, and dependencies between them
- **Agent configuration**: Node labels, Docker agents, Kubernetes pods, or custom agents
- **Post-conditions**: Success/failure/always blocks and their actions (notifications, cleanup, artifact archival)
- **Parameters**: Build parameters (string, choice, boolean, etc.) and their defaults
- **Triggers**: SCM polling, cron schedules, upstream triggers, webhook triggers
- **Environment variables**: Global and stage-level environment variable definitions
- **Conditional logic**: `when` blocks, `if/else` in scripted pipelines, branch-based logic

### Plugin Dependency Inventory

Create a comprehensive inventory of all Jenkins plugins used:

| Jenkins Plugin | Purpose | GHA Equivalent | Migration Notes |
|---|---|---|---|
| [plugin name] | [what it does] | [GHA action or native feature] | [complexity, gotchas] |

Common plugins to look for:
- Credentials Binding, Pipeline Utility Steps, Workspace Cleanup
- Docker Pipeline, Kubernetes, Amazon ECR
- JUnit, Cobertura, HTML Publisher
- Slack Notification, Email Extension
- Git, GitHub, Bitbucket
- SonarQube Scanner, Artifactory, Nexus
- Parameterized Trigger, Build Flow

### Shared Library Analysis

If `@Library` annotations or shared library imports are present:

| Library | Functions Used | Migration Strategy |
|---|---|---|
| [library name] | [list of functions/vars called] | [reusable workflow / composite action / inline] |

For each shared library function:
- Understand what it does (read source if available)
- Determine if it can be replaced by a GitHub Action, reusable workflow, or inline script
- Note any organization-wide dependencies

### Credentials Inventory

Document all credentials referenced in the pipeline:

| Credential ID | Type | Usage Context | Migration Target |
|---|---|---|---|
| [credential-id] | [usernamePassword / secret / sshKey / certificate / token] | [where and how it's used] | [GitHub secret scope: repo/org/environment] |

### Integration Points

Identify all external integrations:
- External services (SonarQube, Artifactory, Nexus, container registries, cloud providers)
- Webhook configurations (incoming and outgoing)
- Artifact storage and retrieval
- Deployment targets (servers, Kubernetes clusters, cloud services)
- Notification channels (Slack, email, Teams)
- Approval gates and manual intervention points

## Step 3: Migration Scope Assessment

Evaluate whether this migration request is appropriately sized for this spec-driven workflow.

**Chain-of-thought reasoning:**

- Consider the complexity of the Jenkins pipeline(s)
- Count plugins, shared libraries, and integration points
- Use context from Step 2 to inform the assessment
- If scope is too large, suggest breaking into smaller migration specs
- If scope is too small, suggest direct implementation without formal spec

**Scope Examples:**

**Too Large (split into multiple migration specs):**

- Organization-wide migration of all Jenkins pipelines
- 10+ pipelines migrated simultaneously
- Migrating pipelines while also changing build tools (e.g., Maven to Gradle)
- Migrating pipelines plus infrastructure provisioning (new runners, new cloud accounts)
- Complete shared library ecosystem migration in one spec

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

## Step 4: Clarifying Questions

Ask clarifying questions to gather sufficient detail for the migration. Focus on understanding both the current state and the desired target state.

Use the following migration-specific focus areas to guide your questions:

**Current State:**

- Which Jenkins plugins are critical vs. nice-to-have?
- Are there shared libraries? Is the source available?
- What integrations exist (artifact repos, cloud providers, notification channels)?
- Are there undocumented manual steps in the current pipeline?

**Target State:**

- Is this a 1:1 translation or an opportunity to improve the pipeline?
- What runner strategy is preferred (GitHub-hosted, self-hosted, larger runners)?
- Should GitHub-native features be adopted (Environments, OIDC, Dependency Review)?
- Are there new requirements not in the current Jenkins pipeline?

**Secrets & Security:**

- How are credentials currently managed (Jenkins credential store, external vault)?
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

**Progressive Disclosure:** Start with Current State and Target State, then expand based on migration complexity and user responses.

### Questions File Format

Follow this format exactly when you create the questions file.

```markdown
# [NN] Questions Round 1 - [Feature Name]

Please answer each question below (select one or more options, or add your own notes). Feel free to add additional context under any question.

## 1. [Question Category/Topic]

[What specific aspect of the feature needs clarification?]

- [ ] (A) [Option description explaining what this choice means]
- [ ] (B) [Option description explaining what this choice means]
- [ ] (C) [Option description explaining what this choice means]
- [ ] (D) [Option description explaining what this choice means]
- [ ] (E) Other (describe)

## 2. [Another Question Category/Topic]

[What specific aspect of the feature needs clarification?]

- [ ] (A) [Option description explaining what this choice means]
- [ ] (B) [Option description explaining what this choice means]
- [ ] (C) [Option description explaining what this choice means]
- [ ] (D) [Option description explaining what this choice means]
- [ ] (E) Other (describe)
```

### Questions File Process

1. **Create Questions File**: Save questions to a file named `[NN]-questions-[N]-[feature-name].md` where `[N]` is the round number (starting at 1, incrementing for each new round).
2. **Point User to File**: Direct the user to the questions file and instruct them to answer the questions directly in the file.
3. **STOP AND WAIT**: Do not proceed to Step 5. Wait for the user to indicate they have saved their answers.
4. **Read Answers**: After the user indicates they have saved their answers, read the file and continue the conversation.
5. **Follow-Up Rounds**: If answers reveal new questions, create a new questions file with incremented round number (`[NN]-questions-[N+1]-[feature-name].md`) and repeat the process (return to step 3).

**Iterative Process:**

- If a user's answer reveals new questions or areas needing clarification, ask follow-up questions in a new questions file.
- Build on previous answers - use context from earlier responses to inform subsequent questions.
- **CRITICAL**: After creating any questions file, you MUST STOP and wait for the user to provide answers before proceeding.
- Only proceed to Step 5 after:
  - You have received and reviewed all user answers to clarifying questions
  - You have enough detail to populate all spec sections (Pipeline Assessment, Platform Deltas, Secrets Strategy, Cutover Plan, etc.).

## Step 5: Migration Spec Generation

Generate a comprehensive migration specification using this exact structure:

```markdown
# [NN]-spec-[feature-name].md

## Introduction/Overview

[Briefly describe the migration scope and the problem it solves. State what Jenkins pipeline(s) are being migrated and why. 2-3 sentences.]

## Migration Goals

[List 3-5 specific, measurable objectives for this migration. Use bullet points.]

- Achieve functional parity with the existing Jenkins pipeline
- [Goal 2: e.g., Reduce build times by leveraging parallel GHA jobs]
- [Goal 3: e.g., Eliminate Jenkins plugin dependency risk]
- [Goal 4: e.g., Adopt OIDC for cloud authentication]
- [Goal 5: e.g., Enable self-service pipeline modifications via PR-based workflow changes]

## Current Jenkins Configuration

### Pipeline Overview

- **Pipeline Type**: [Declarative / Scripted / Multibranch]
- **Trigger(s)**: [SCM poll / cron / webhook / upstream]
- **Agent Strategy**: [Node label / Docker / Kubernetes / any]
- **Parameters**: [List build parameters]
- **Estimated Build Duration**: [Typical build time]

### Stage Inventory

| Stage Name | Purpose | Key Actions | Dependencies | Post-Conditions |
|---|---|---|---|---|
| [stage] | [what it does] | [commands/plugins used] | [prior stages] | [success/failure actions] |

### Plugin Dependencies

| Jenkins Plugin | Purpose | GHA Equivalent | Migration Complexity | Notes |
|---|---|---|---|---|
| [plugin] | [purpose] | [action/feature] | [Low/Medium/High] | [gotchas] |

### Shared Libraries

| Library | Functions Used | Current Behavior | Migration Strategy |
|---|---|---|---|
| [library] | [functions] | [what they do] | [reusable workflow / composite action / inline] |

### Credentials Inventory

| Credential ID | Type | Used In Stage(s) | GHA Migration Target | Rotation Required |
|---|---|---|---|---|
| [id] | [type] | [stages] | [repo secret / org secret / environment secret / OIDC] | [Yes/No] |

## Target GitHub Actions Architecture

### Workflow Structure

[Describe the target workflow file(s), their triggers, and how they map to the current Jenkins stages. Include whether single or multiple workflow files are needed and why.]

### Runner Strategy

[Specify runner types: GitHub-hosted (ubuntu-latest, windows-latest), larger runners, or self-hosted. Justify the choice based on build requirements.]

### Environment Strategy

[Describe how GitHub Environments will be used for deployment stages, including protection rules, required reviewers, and environment-specific secrets.]

## Platform Delta Analysis

Map Jenkins concepts to their GitHub Actions equivalents, noting approach and risk:

| Jenkins Concept | GHA Equivalent | Approach | Risk Level |
|---|---|---|---|
| [concept] | [equivalent] | [how to migrate] | [Low/Medium/High] |

**Common deltas to evaluate:**

- [ ] `agent` / `node` → `runs-on` (runner selection)
- [ ] `stage` → `job` or `step` (granularity decision)
- [ ] `post { always/success/failure }` → `if: always()` / `if: success()` / `if: failure()`
- [ ] `parameters` → `workflow_dispatch` inputs or `workflow_call` inputs
- [ ] `when { branch }` → `on: push: branches:` or job-level `if:`
- [ ] `parallel` → `strategy.matrix` or multiple jobs
- [ ] `stash/unstash` → `actions/upload-artifact` + `actions/download-artifact`
- [ ] `sh` / `bat` → `run:` steps
- [ ] `withCredentials` → `secrets` context + environment variables
- [ ] `input` (manual approval) → Environment protection rules with required reviewers
- [ ] `timeout` → `timeout-minutes` at job or step level
- [ ] `retry` → custom retry logic or action (no native equivalent)
- [ ] `lock` (Lockable Resources) → concurrency groups
- [ ] Shared Libraries → Reusable workflows or composite actions
- [ ] `currentBuild.result` → `${{ job.status }}` or `outcome`/`conclusion`
- [ ] `Jenkinsfile` in repo root → `.github/workflows/*.yml`
- [ ] Multibranch Pipeline → Multiple `on:` trigger configurations

## Secrets Migration Strategy

### Scoping Plan

| Secret | Current Jenkins Scope | Target GHA Scope | Justification |
|---|---|---|---|
| [secret] | [global / folder / pipeline] | [repo / org / environment] | [why this scope] |

### OIDC Federation

[Describe OIDC federation plan for cloud providers (AWS, Azure, GCP). If not applicable, state why and document the alternative approach.]

### Rotation Plan

[Document which secrets need rotation during migration, the rotation procedure, and validation steps.]

### Validation Approach

[How will you verify all secrets are correctly configured before cutover? Include test procedures.]

## Migration Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| [risk description] | [Low/Medium/High] | [Low/Medium/High] | [mitigation strategy] |

**Common risks to evaluate:**

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

[Focus on tangible progress and WHAT will be demonstrated. Define 2-4 small, end-to-end vertical slices using the format below. Each unit should produce a working workflow that can be validated independently.]

### [Unit 1]: [Title]

**Purpose:** [What this slice accomplishes — e.g., "Core build and test stages migrated with artifact publishing"]

**Functional Requirements:**
- The workflow shall [requirement 1: clear, testable, unambiguous]
- The workflow shall [requirement 2: clear, testable, unambiguous]

**Proof Artifacts:**
- [Artifact type]: [description] demonstrates [what it proves]
- Example: `GHA Run: Successful workflow run on feature branch demonstrates build parity`
- Example: `Artifact: Build output matches Jenkins artifact demonstrates functional equivalence`

### [Unit 2]: [Title]

**Purpose:** [What this slice accomplishes]

**Functional Requirements:**
- The workflow shall [requirement 1: clear, testable, unambiguous]
- The workflow shall [requirement 2: clear, testable, unambiguous]

**Proof Artifacts:**
- [Artifact type]: [description] demonstrates [what it proves]

## Non-Goals (Out of Scope)

[Clearly state what this migration will NOT include to manage expectations and prevent scope creep.]

**Common non-goals to consider:**

- Rewriting application build logic (only migrating the CI/CD pipeline)
- Changing test frameworks or test coverage requirements
- Migrating non-pipeline Jenkins jobs (freestyle, matrix projects)
- Decommissioning the Jenkins instance (separate effort)
- Changing branching strategy
- Migrating Jenkins job configuration history or build logs

1. [**Specific exclusion 1**: description]
2. [**Specific exclusion 2**: description]
3. [**Specific exclusion 3**: description]

## Cutover Strategy

### Parallel Run Period

[Describe the period during which both Jenkins and GitHub Actions will run simultaneously. Include duration, success criteria for ending the parallel run, and how discrepancies will be handled.]

### Validation Criteria

[Define the specific criteria that must be met before Jenkins can be decommissioned for this pipeline:]

- [ ] All stages produce equivalent results
- [ ] Artifact outputs match
- [ ] Deployment targets receive identical artifacts
- [ ] Notification channels receive equivalent alerts
- [ ] Build times are within acceptable range
- [ ] All environment-specific deployments validated

### Rollback Plan

[Document how to revert to Jenkins if the migration encounters critical issues. Include specific steps and decision criteria for triggering rollback.]

### Communication Plan

[Who needs to be informed about the migration, at what stages, and through what channels.]

## Technical Considerations

[Focus on implementation constraints and HOW the migration will be executed.]

- **Runner networking**: [VPN, firewall rules, private network access requirements]
- **GitHub Actions limits**: [Workflow run time limits, job matrix limits, artifact storage limits, API rate limits]
- **Monorepo considerations**: [Path filters, workflow file organization, shared dependencies]
- **Caching strategy**: [What to cache, cache key strategy, cache size limits]
- **Concurrency controls**: [Concurrency groups, queue behavior, cancel-in-progress settings]

## Security Considerations

- **Log masking**: Ensure all secrets are masked in workflow logs using `::add-mask::` where needed
- **Branch protection**: Configure required status checks, enforce PR reviews for workflow file changes
- **Action pinning**: All third-party actions MUST be pinned to a full SHA (not a tag or branch)
- **GITHUB_TOKEN scoping**: Use minimum required permissions with explicit `permissions:` block
- **Fork PR security**: Configure `pull_request_target` carefully; never checkout fork code in privileged context
- **Workflow permissions**: Use `permissions: read-all` as default, grant write only where needed
- [Additional project-specific security considerations]

## Success Metrics

[How will migration success be measured? Include specific metrics where possible.]

1. [**Functional parity**: All Jenkins stages have equivalent GHA jobs/steps producing same results]
2. [**Build time**: GHA build time within X% of Jenkins build time]
3. [**Reliability**: GHA workflow success rate matches or exceeds Jenkins]
4. [**Security**: All secrets migrated with appropriate scoping, OIDC where applicable]
5. [**Cutover**: Parallel run period completed with zero discrepancies]

## Open Questions

[List any remaining questions or areas needing clarification. If none, state "No open questions at this time."]

1. [Question 1]
2. [Question 2]
```

## Step 6: Review and Refinement

After generating the migration spec, present it to the user and ask:

1. "Does the Jenkins pipeline assessment accurately capture all stages, plugins, and integrations?"
2. "Is the platform delta analysis complete — are there any Jenkins behaviors not accounted for?"
3. "Is the secrets migration strategy appropriate for your security requirements?"
4. "Is the cutover strategy realistic given your team's capacity and risk tolerance?"
5. "Are the demoable units sized appropriately for incremental validation?"

Iterate based on feedback until the user is satisfied.

## Output Requirements

**Format:** Markdown (`.md`)
**Full Path:** `./docs/specs/[NN]-spec-[feature-name]/[NN]-spec-[feature-name].md`
**Example:** For migrating a build pipeline, the spec directory would be `01-spec-migrate-build-pipeline/` with a spec file as `01-spec-migrate-build-pipeline.md` inside it

## Critical Constraints

**NEVER:**

- Start implementing the migration; only create the specification document
- Recommend unpinned third-party actions (always require full SHA pinning)
- Skip the Jenkins pipeline assessment — it is the foundation of the migration
- Assume plugin equivalents exist without researching them
- Create specs that are too large or too small without addressing scope issues
- Skip the clarifying questions phase, even if the pipeline seems straightforward
- Omit the platform delta analysis
- Omit the secrets migration strategy
- Omit the risk assessment
- Omit the cutover strategy

**ALWAYS:**

- Ask clarifying questions before generating the spec
- Validate scope appropriateness before proceeding
- Use the exact spec structure provided above
- Include a complete plugin inventory with GHA equivalents
- Include platform delta analysis mapping all Jenkins concepts to GHA equivalents
- Include secrets migration strategy with OIDC recommendations where applicable
- Include risk assessment with mitigation strategies
- Include cutover strategy with rollback plan
- Include proof artifacts for each demoable unit
- Pin all referenced third-party actions to full commit SHAs

## What Comes Next

Once this migration spec is complete and approved, instruct the user to run `/SDM-2-generate-migration-tasks`. This will start the next step in the workflow, which is to break down the migration specification into actionable tasks.
