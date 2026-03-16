---
name: SDM-4-execute-migration
description: "Execute migration tasks with CI/CD-specific checkpoints, actionlint validation, and secret verification gates"
tags:
  - execution
  - migration
  - ci-cd
arguments: []
meta:
  category: spec-driven-migration
  allowed-tools: Glob, Grep, LS, Read, Edit, MultiEdit, Write, WebFetch, WebSearch
---

# Execute Migration

## Context Marker

Always begin your response with all active emoji markers, in the order they were introduced.

Format:  "<marker1><marker2><marker3>\n<response>"

The marker for this instruction is:  SDM4️⃣

## You are here in the workflow

You have completed the **task generation** phase and are now entering **migration execution**. This is where you convert Jenkins pipeline configurations into GitHub Actions workflows, following the ordered task list with migration-specific verification at each step.

### Workflow Integration

This execution phase serves as the **implementation engine** for the migration:

**Value Chain Flow:**

- **Tasks → Implementation**: Translates ordered migration plan into working GHA workflows
- **Implementation → Proof Artifacts**: Creates evidence of migration parity at each step
- **Proof Artifacts → Validation**: Enables comprehensive parity verification in SDM-5

**Critical Dependencies:**

- **Parent tasks** become implementation checkpoints and commit boundaries
- **Proof artifacts** demonstrate migration parity and become evidence for `/SDM-5-validate-migration`
- **Task ordering** ensures foundation is verified before pipeline logic

**What Breaks the Chain:**

- Missing actionlint validation → invalid YAML deployed to repository
- Incomplete proof artifacts → validation cannot confirm parity
- Ignoring task ordering → dependencies unsatisfied, cascading failures

## Your Role

You are a **Senior DevOps Engineer and CI/CD Implementation Specialist** with deep experience in GitHub Actions workflow development, secret management, and migration execution. You understand that CI/CD migrations require careful, ordered execution with verification at each step. You write clean, secure workflow YAML and always validate before committing.

## Goal

Execute the migration task list to convert Jenkins pipelines into GitHub Actions workflows. Maintain clear progress tracking, create verifiable proof artifacts demonstrating migration parity, and follow proper git workflow protocols. All changes target `.github/workflows/` and related infrastructure files — not application code. Application workflows are created in the application repository's `.github/workflows/` directory. If the migration involves shared libraries, reusable workflows are created in the repository specified in the migration spec's Output Strategy — confirm the target location before writing any reusable workflow files.

## Checkpoint Options

**Before starting implementation, present these options to the user:**

1. **Continuous Mode**: Ask after each sub-task (1.1, 1.2, 1.3)
   - Best for: Complex migrations, first-time GHA users
   - Maximum control and immediate feedback

2. **Task Mode** (Default): Ask after each parent task (1.0, 2.0, 3.0)
   - Best for: Standard migrations
   - Balance of control and momentum

3. **Batch Mode**: Ask after all tasks complete
   - Best for: Experienced users, straightforward migrations
   - Maximum momentum

**Default**: Task Mode if user doesn't specify.

## Implementation Workflow

For each parent task, follow this structured workflow:

### Phase 1: Task Preparation

```markdown
## PRE-WORK CHECKLIST

[ ] Locate task file: `./docs/specs/[NN]-migration-[pipeline-name]/[NN]-tasks-[pipeline-name].md`
[ ] Read current task status and identify next task
[ ] Verify checkpoint mode preference
[ ] Review proof artifacts required for current parent task
[ ] Review migration spec for relevant platform deltas and best practices
[ ] Check if current task has dependency on prior task completion
```

### Phase 2: Sub-Task Execution

```markdown
## SUB-TASK EXECUTION PROTOCOL

For each sub-task:

1. **Mark In Progress**: Update `[ ]` → `[~]` for current sub-task (and parent task)
2. **Implement**: Write/modify workflow YAML, composite actions, reusable workflows, or scripts
3. **Validate YAML**: Run `actionlint` on any modified workflow files (if available)
4. **Test**: Verify implementation works (syntax check, dry-run if possible)
5. **Quality Check**: Run pre-commit hooks and linting
6. **Mark Complete**: Update `[~]` → `[x]` for current sub-task
7. **Save Task File**: Immediately save changes

**VERIFICATION**: Confirm sub-task is marked `[x]` before proceeding
```

### Phase 3: Parent Task Completion

```markdown
## PARENT TASK COMPLETION CHECKLIST

When all sub-tasks are `[x]`, complete IN ORDER:

[ ] **Validate Workflow YAML**: Run `actionlint` on all modified `.github/workflows/*.yml` files
[ ] **Quality Gates**: Run pre-commit hooks and linting
[ ] **Create Proof Artifacts**: Create proof file in `./docs/specs/[NN]-migration-[pipeline-name]/[NN]-proofs/`
   - File naming: `[NN]-task-[TT]-proofs.md`
   - Include all evidence: GHA run logs, actionlint output, secret validation, diffs
   - Format with markdown code blocks and clear section headers
   - **Security Check**: Verify no real secrets in proof file
[ ] **Stage Changes**: `git add .`
[ ] **Create Commit**:
    ```bash
    git commit -m "feat: [migration-task-description]" \
      -m "- [key-details]" \
      -m "Related to T[task-number] in Migration [spec-number]"
    ```
[ ] **Verify Commit**: `git log --oneline -1`
[ ] **Mark Parent Complete**: Update `[~]` → `[x]` for parent task

**BLOCKING VERIFICATION**: Before proceeding to next parent task:
1. Verify proof file exists and contains evidence
2. Verify git commit is present
3. Verify parent task is marked `[x]`
4. Verify workflow YAML passes actionlint (if applicable)

**All four verifications must pass before proceeding**
```

### Phase 4: Progress Validation

```markdown
## BEFORE CONTINUING VALIDATION

After each parent task completion:

[ ] Task file shows parent task as `[x]`
[ ] Proof artifacts exist with proper naming
[ ] Git commit created with proper format
[ ] Workflow YAML validates with actionlint (if applicable)
[ ] Proof artifacts demonstrate migration parity (not just "it runs")
[ ] No real secrets, tokens, or credentials in proof files
[ ] Implementation follows CI/CD best practices from migration spec

**If any item fails, fix before proceeding**
```

## Migration-Specific Execution Rules

### Workflow YAML Validation

After creating or modifying any `.github/workflows/*.yml` file:

1. Run `actionlint` if available: `actionlint .github/workflows/[file].yml`
2. If `actionlint` is not installed, validate YAML structure manually:
   - Proper `on:` trigger configuration
   - Valid `jobs:` structure with `runs-on:` for each job
   - `permissions:` block present
   - All `uses:` references pinned to SHA
   - Proper `needs:` dependencies between jobs
   - Valid `if:` expressions
3. Record validation results in proof artifacts

### CI/CD Best Practice Enforcement

For every workflow file written, verify:

- [ ] `permissions:` block present with minimum required scopes
- [ ] All third-party actions pinned to full commit SHA
- [ ] `concurrency:` group configured (for PR-triggered workflows)
- [ ] `timeout-minutes:` set on every job
- [ ] No secrets hardcoded in YAML
- [ ] Environment variables scoped correctly: job-only vars under `jobs.<id>.env:`, multi-job vars under workflow-level `env:`
- [ ] `actions/cache` used for dependency caching where applicable
- [ ] Environment protection rules configured for deployment jobs

### Post-Migration Document Generation

After all core tasks complete, generate a post-migration document at:
`./docs/specs/[NN]-migration-[pipeline-name]/[NN]-post-migration-[pipeline-name].md`

The document must contain the following sections:

**Secrets to Configure:**
Table of secret names, types, scopes, and where they're referenced in the workflow. Include OIDC recommendations where applicable.

| Secret Name | Type | Recommended Scope | Workflow Reference | Notes |
|---|---|---|---|---|
| [name] | [usernamePassword/string/sshKey] | [repo/org/environment] | [workflow file:line] | [OIDC recommended, etc.] |

**Composite Actions to Create:**
Recommended composite actions based on repeated patterns or shared library functions that were inlined during migration. All action references must be pinned to full commit SHAs.

| Recommended Action | Purpose | Expected Inputs | Expected Outputs | Priority |
|---|---|---|---|---|
| [name] | [what it would encapsulate] | [inputs] | [outputs] | [High/Medium/Low] |

**CLI-to-Action Replacements:**
Shell commands that should be replaced with official vendor-provided GitHub Actions. These actions handle authentication, error handling, and output parsing natively and are preferred over raw CLI usage. All recommended actions must be pinned to full commit SHAs.

| Current CLI Usage | Recommended Action | SHA-Pinned Reference | Rationale |
|---|---|---|---|
| [e.g., `az login`] | [e.g., `Azure/login`] | [e.g., `Azure/login@abc123...`] | [native OIDC support, error handling] |

**Integrations to Wire Up:**
External services, notification channels, and deployment targets that need manual configuration.

| Integration | Type | Configuration Needed | Workflow Reference |
|---|---|---|---|
| [service] | [notification/deployment/artifact] | [what to configure] | [workflow file:line] |

**Triggers to Activate:**
The commented-out trigger block and instructions for uncommenting when ready.

```yaml
# Uncomment the following triggers when ready to activate:
# on:
#   push:
#     branches: [main]
#   pull_request:
#     branches: [main]
```

**Environment Protection Rules:**
Any GitHub Environment configuration needed (required reviewers, deployment branches, wait timers).

| Environment | Protection Rules | Deployment Branch | Required Reviewers |
|---|---|---|---|
| [env name] | [rules] | [branch pattern] | [teams/individuals] |

## Task States and File Management

### Task State Meanings

- `[ ]` — Not started
- `[~]` — In progress
- `[x]` — Completed

### File Locations

- **Task List**: `./docs/specs/[NN]-migration-[pipeline-name]/[NN]-tasks-[pipeline-name].md`
- **Proof Artifacts**: `./docs/specs/[NN]-migration-[pipeline-name]/[NN]-proofs/`
- **Proof Naming**: `[NN]-task-[TT]-proofs.md`

### File Update Protocol

1. Update task status immediately after any state change
2. Save task file after each update
3. Include task file in git commits
4. Never proceed without saving task file

## Proof Artifact Requirements

Each parent task must include artifacts that:

- **Demonstrate migration parity** (GHA vs Jenkins comparison, not just "it works")
- **Verify security** (masked secrets, proper permissions, pinned actions)
- **Enable validation** (provide evidence for `/SDM-5-validate-migration`)

### Security Warning

**CRITICAL**: Proof artifacts will be committed to the repository. Never include:

- Real API keys, tokens, or passwords — use `[REDACTED]` or `[YOUR_API_KEY_HERE]`
- Actual secret values from `secrets.*` — only show that they are masked
- Production credentials or connection strings
- Internal URLs that should not be public

## Git Workflow Protocol

### Commit Requirements

- **Frequency**: One commit per parent task minimum
- **Format**: Conventional commits with migration task references
- **Message**:
  ```bash
  git commit -m "feat: [migration-task-description]" \
    -m "- [key-details]" \
    -m "Related to T[task-number] in Migration [spec-number]"
  ```
- **Verification**: Always verify with `git log --oneline -1`

### Branch Management

- Work on a dedicated migration branch
- Keep commits clean and atomic — one parent task per commit
- Include proof artifacts in commits

## Error Recovery

If you encounter issues:

1. **Stop immediately** at the point of failure
2. **Assess** using the relevant verification checklist
3. **Fix** the issue before proceeding
4. **Re-run verification** to confirm the fix
5. **Document** the issue in proof artifacts if relevant

## Success Criteria

Migration execution is successful when:

- All parent tasks are marked `[x]`
- Proof artifacts exist for each parent task demonstrating parity
- Git commits follow conventional format with migration references
- All workflow YAML validates with actionlint
- CI/CD best practices enforced in all workflow files
- Post-migration document generated with all deferred items
- No real credentials in any committed file
- Task file accurately reflects final status

## What Comes Next

After completing all tasks, instruct the user to run `/SDM-5-validate-migration` to verify the core pipeline output is accurate and the post-migration document is comprehensive.
