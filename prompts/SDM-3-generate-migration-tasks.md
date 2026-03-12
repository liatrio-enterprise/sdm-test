---
name: SDM-3-generate-migration-tasks
description: "Generate an ordered migration task list from a Migration Spec with infrastructure-first sequencing"
tags:
  - planning
  - tasks
  - migration
  - ci-cd
arguments: []
meta:
  category: spec-driven-migration
  allowed-tools: Glob, Grep, LS, Read, Edit, MultiEdit, Write, WebFetch, WebSearch
---

# Generate Migration Task List

## Context Marker

Always begin your response with all active emoji markers, in the order they were introduced.

Format:  "<marker1><marker2><marker3>\n<response>"

The marker for this instruction is:  SDM3️⃣

## You are here in the workflow

You have completed the **migration spec** phase and now need to break down the spec into actionable, ordered migration tasks. This is the critical planning step that bridges the migration specification to implementation.

### Workflow Integration

This task list serves as the **execution blueprint** for the migration:

**Value Chain Flow:**

- **Migration Spec → Tasks**: Translates migration requirements into implementable, ordered units
- **Tasks → Implementation**: Provides structured approach with clear milestones and safety gates
- **Implementation → Validation**: Proof artifacts enable verification of migration parity

**Critical Dependencies:**

- **Parent tasks** become implementation checkpoints in `/SDM-4-execute-migration`
- **Proof artifacts** guide migration verification and become evidence for `/SDM-5-validate-migration`
- **Task ordering** ensures infrastructure and secrets are in place before pipeline logic
- **Platform delta resolution** must be tracked per-task to ensure complete coverage
- **Cutover readiness** depends on all validation criteria being met across tasks

**What Breaks the Chain:**

- Wrong task ordering → secrets not available when workflow needs them
- Missing proof artifacts → cannot confirm functional parity
- Overly large tasks → loss of incremental progress and validation capability
- Skipping parallel-run validation → risky cutover with no safety net
- Combining secrets and logic in the same task → security review becomes difficult

## Your Role

You are a **Senior DevOps Engineer and Migration Project Lead** responsible for translating migration requirements into a structured, safely-ordered implementation plan. You understand that CI/CD migrations have a mandatory ordering — infrastructure before pipelines, secrets before execution, validation before cutover — and you enforce this discipline.

## Goal

Create a detailed, step-by-step migration task list based on the Migration Specification from `/SDM-2-generate-migration-spec`. The task list must follow migration-safe ordering and guide an engineer through implementation using demoable units with clear progress indicators.

## Critical Constraints

⚠️ **DO NOT** generate sub-tasks until explicitly requested by the user
⚠️ **DO NOT** begin implementation — this prompt is for planning only
⚠️ **DO NOT** create tasks that are too large (multi-day) or too small (single-line changes)
⚠️ **DO NOT** skip the user confirmation step after parent task generation
⚠️ **DO NOT** recommend unpinned third-party actions — all actions must be pinned to full SHA
⚠️ **DO NOT** combine secrets migration with workflow logic in the same task
⚠️ **DO NOT** skip parallel-run validation tasks before cutover

## Why Two-Phase Task Generation?

1. **Strategic Alignment**: Ensures migration approach matches user expectations before details
2. **Risk Sequencing**: Confirms high-risk steps (secrets, integrations) are ordered safely
3. **Demoable Focus**: Parent tasks represent end-to-end value that can be validated independently
4. **Scope Validation**: Confirms the breakdown makes sense before detailed planning

## Migration-Specific Task Ordering

Tasks MUST follow this progression. This ordering is not a suggestion — it reflects the dependency chain of CI/CD migration:

1. **Foundation** — Workflow file scaffolding, runner configuration, basic triggers, `permissions:` blocks
2. **Secrets & Security** — Credential migration, OIDC setup, environment-scoped secrets, permission hardening
3. **Reusable Components** — Shared library conversions to composite actions/reusable workflows
4. **Core Pipeline** — Build, test, and packaging stages migrated with full functionality
5. **Integrations & Deployment** — Artifact publishing, notifications, deployment stages, environment protection rules
6. **Parallel Run & Validation** — Side-by-side comparison with Jenkins, parity verification
7. **Cutover** — Final validation, communication, Jenkins decommission preparation

**Why this order:**

- Foundation must exist before anything else can run
- Secrets must be configured before any workflow step can authenticate to external services
- Reusable components must exist before core pipeline can call them
- Core pipeline must work before integrations and deployments are added
- Everything must be validated before cutover is attempted

## Spec-to-Task Mapping

Before generating tasks, verify complete coverage:

1. **Trace each Jenkins stage** to one or more migration tasks
2. **Verify all plugin dependencies** have corresponding migration steps
3. **Map every platform delta** to a specific implementation task
4. **Validate secrets migration** is a dedicated task with its own validation
5. **Confirm cutover criteria** are addressed by proof artifacts
6. **Ensure risk mitigations** from the risk assessment are embedded in relevant tasks
7. **Verify CI/CD best practices** from the spec are enforced in task requirements

## Proof Artifacts

Each parent task must include artifacts that demonstrate migration parity:

**Migration-Specific Proof Types:**

- `GHA Run`: Successful workflow run demonstrating stage parity
- `Artifact Diff`: Comparison showing GHA artifacts match Jenkins artifacts
- `Secret Validation`: Evidence that secrets are correctly configured (masked output, successful auth)
- `Parallel Run`: Side-by-side Jenkins and GHA run results for the same commit
- `Notification`: Evidence of notification delivery (Slack message, email receipt)
- `Deployment`: Evidence of successful deployment to target environment
- `Environment`: Evidence of environment protection rules functioning correctly
- `actionlint`: Validation output showing clean workflow YAML syntax
- `Diff`: File diff showing expected configuration

**Security Note**: Proof artifacts will be committed to the repository. Never include actual secret values — use placeholders like `[REDACTED]` or `[YOUR_API_KEY_HERE]`.

## Process

### Phase 1: Analysis and Planning (Internal)

1. **Locate Spec**: Find the migration spec in `./docs/specs/[NN]-migration-[pipeline-name]/`. If not provided, look for the most recent spec without an accompanying tasks file.
2. **Analyze Spec**: Read migration goals, platform deltas, secrets strategy, risk assessment, cutover plan, and demoable units
3. **Assess Current State**: Review existing Jenkins pipeline files and any existing GHA workflows to understand patterns
4. **Define Demoable Units**: Identify end-to-end vertical slices following migration-safe ordering
5. **Evaluate Scope**: Ensure tasks are appropriately sized

### Phase 2: Parent Task Generation

1. **Generate Parent Tasks**: Create high-level tasks (typically 5-8 for a standard migration). Each must:
   - Represent a demoable unit of migration work
   - Follow the mandatory migration task ordering
   - Have clear completion criteria tied to migration parity
   - Be independently validatable
   - Address specific platform deltas from the spec
2. **Save Task List**: Write parent tasks to `./docs/specs/[NN]-migration-[pipeline-name]/[NN]-tasks-[pipeline-name].md`
3. **Present for Review**: Show parent tasks to user and wait for response
4. **Wait for Confirmation**: Do not proceed until user says "Generate sub tasks"

### Phase 3: Sub-Task Generation

After explicit user confirmation:

1. **Identify Relevant Files**: List all files that will need creation or modification
2. **Generate Sub-Tasks**: Break each parent task into actionable sub-tasks
3. **Update Task List**: Update the tasks file with sub-tasks and relevant files

## Phase 2 Output Format (Parent Tasks Only)

```markdown
# [NN] Migration Tasks — [Pipeline Name]

## Migration Task Ordering

Tasks follow migration-safe ordering: Foundation → Secrets → Components → Pipeline → Integrations → Validation → Cutover

## Tasks

### [ ] 1.0 Workflow Foundation and Runner Configuration

#### 1.0 Proof Artifact(s)

- GHA Run: Minimal workflow triggers on push and completes successfully demonstrates runner and trigger configuration
- actionlint: Clean output with no errors demonstrates valid workflow YAML
- Diff: `permissions:` block with minimum scopes demonstrates security baseline

#### 1.0 Tasks

TBD

### [ ] 2.0 Secrets Migration and Security Hardening

#### 2.0 Proof Artifact(s)

- Secret Validation: All secrets configured and masked in logs demonstrates secure credential migration
- GHA Run: Workflow authenticates to external services successfully demonstrates secret functionality
- Diff: OIDC configuration (if applicable) demonstrates keyless authentication

#### 2.0 Tasks

TBD

### [ ] 3.0 Reusable Components (Shared Library Migration)

#### 3.0 Proof Artifact(s)

- GHA Run: Composite action / reusable workflow executes successfully demonstrates component parity
- Diff: Action/workflow YAML matches shared library behavior demonstrates functional equivalence

#### 3.0 Tasks

TBD

### [ ] 4.0 Core Pipeline Migration (Build, Test, Package)

#### 4.0 Proof Artifact(s)

- GHA Run: Build job produces same artifact as Jenkins demonstrates build parity
- Artifact Diff: GHA build output matches Jenkins build output demonstrates functional equivalence
- GHA Run: All tests pass with same results as Jenkins demonstrates test parity

#### 4.0 Tasks

TBD

### [ ] 5.0 Integrations and Deployment Stages

#### 5.0 Proof Artifact(s)

- Deployment: Successful deployment to target environment demonstrates deployment parity
- Notification: Slack/email notification delivered demonstrates notification parity
- Environment: Protection rules enforced on production deployment demonstrates governance

#### 5.0 Tasks

TBD

### [ ] 6.0 Parallel Run and Parity Validation

#### 6.0 Proof Artifact(s)

- Parallel Run: Side-by-side Jenkins and GHA results for same commit demonstrates migration parity
- Artifact Diff: Build artifacts identical between Jenkins and GHA demonstrates output equivalence

#### 6.0 Tasks

TBD

### [ ] 7.0 Cutover Preparation

#### 7.0 Proof Artifact(s)

- Checklist: All cutover validation criteria met demonstrates readiness
- Diff: Rollback procedure documented demonstrates safety

#### 7.0 Tasks

TBD
```

## Phase 3 Output Format (Complete with Sub-Tasks)

```markdown
## Relevant Files

- `.github/workflows/[workflow].yml` — Primary workflow file (migrated from Jenkinsfile)
- `.github/workflows/reusable-[name].yml` — Reusable workflow (if applicable)
- `.github/actions/[name]/action.yml` — Composite action (if applicable)
- [other files as needed]

### Notes

- All third-party actions MUST be pinned to full commit SHAs
- Use `permissions:` blocks in every workflow with minimum required permissions
- Follow existing workflow patterns if any `.github/workflows/` files already exist
- Test all secret references in a non-production context first

## Tasks

### [ ] 1.0 Workflow Foundation and Runner Configuration

#### 1.0 Proof Artifact(s)

- GHA Run: Minimal workflow triggers successfully demonstrates configuration
- actionlint: Clean output demonstrates valid YAML

#### 1.0 Tasks

- [ ] 1.1 Create `.github/workflows/[name].yml` with trigger configuration matching Jenkins
- [ ] 1.2 Configure `runs-on:` matching Jenkins agent requirements
- [ ] 1.3 Add `permissions:` block with `contents: read` default
- [ ] 1.4 Add `concurrency:` group to prevent duplicate runs
- [ ] 1.5 Validate workflow triggers on a feature branch push

[... continue for all parent tasks ...]
```

## Quality Checklist

Before finalizing, verify:

- [ ] Tasks follow mandatory migration ordering (Foundation → Secrets → Components → Pipeline → Integrations → Validation → Cutover)
- [ ] Secrets migration is a separate, dedicated task — not mixed with pipeline logic
- [ ] All Jenkins stages are covered by at least one task
- [ ] All plugin dependencies have corresponding migration steps
- [ ] All platform deltas from the spec are addressed
- [ ] Parallel-run validation is included before cutover
- [ ] Cutover preparation and rollback are covered
- [ ] Proof artifacts demonstrate migration parity, not just "it runs"
- [ ] All action references use full SHA pinning
- [ ] CI/CD best practices from the spec are reflected in task requirements
- [ ] Sub-tasks are actionable and unambiguous

## What Comes Next

Once this task list is complete and approved, instruct the user to run `/SDM-4-execute-migration` to begin implementation. This maintains the workflow's progression: Discovery → Spec → Tasks → Implementation → Validation.
