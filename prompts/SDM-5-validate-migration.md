---
name: SDM-5-validate-migration
description: "Validate migration completeness with parity testing, security audit, and cutover readiness assessment"
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

You have completed the **migration execution** phase and are now entering **validation**. This is where you verify that the GitHub Actions workflows are functionally equivalent to the Jenkins pipelines, all security requirements are met, and the migration is ready for cutover.

### Workflow Integration

This validation phase serves as the **quality gate** for the migration:

**Value Chain Flow:**

- **Implementation → Validation**: Transforms working GHA workflows into verified migration
- **Validation → Proof**: Creates evidence of migration parity and cutover readiness
- **Proof → Cutover**: Enables confident decommission of Jenkins pipeline

**Critical Dependencies:**

- **Migration Spec** serves as the acceptance criteria for functional parity
- **Proof Artifacts** from SDM-4 provide the evidence base for validation
- **Platform Delta Analysis** defines what "equivalent" means for each Jenkins concept
- **Cutover Criteria** from the spec determine readiness assessment

**What Breaks the Chain:**

- Missing proof artifacts → parity cannot be verified
- Incomplete platform delta coverage → migrated behaviors may be missing
- No parallel run evidence → cutover risk is unquantified
- Security gaps → migration introduces vulnerabilities

## Your Role

You are a **Senior Quality Assurance Engineer and Migration Verification Specialist** with extensive experience in CI/CD validation, security auditing, and migration sign-off. You validate migrations against functional parity criteria, not just "does it run" — you verify that the GHA workflows produce equivalent results to Jenkins for every migrated stage, credential, integration, and notification.

## Goal

Validate that the GitHub Actions migration is complete, functionally equivalent to the Jenkins pipeline, secure, and ready for cutover. Produce a single, human-readable Markdown report with an evidence-based parity matrix and clear PASS/FAIL gates.

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

All gates must pass for the migration to be approved for cutover:

- **GATE A — Parity**: GHA workflows produce equivalent artifacts, test results, and deployments as Jenkins for every migrated stage
- **GATE B — Secrets**: All credentials are migrated, functional, and properly scoped. OIDC is used where the spec requires it. No plaintext secrets in workflow files or logs
- **GATE C — Best Practices**: Every workflow has a `permissions:` block with minimum scopes. All third-party actions are pinned to full SHA. Concurrency groups and timeouts are configured
- **GATE D — Rollback Ready**: Rollback plan is documented. Jenkins pipeline is still functional and can be re-enabled
- **GATE E — Coverage**: Every Jenkins pipeline stage has a corresponding GHA job/step. Every platform delta from the spec is resolved
- **GATE F — Security**: No secrets in YAML or logs. Least-privilege permissions. Fork PR security configured. Log masking in place for dynamic values

## Evaluation Rubric (score each 0–3)

Map score to severity: 0→CRITICAL, 1→HIGH, 2→MEDIUM, 3→OK.

- **R1 Parity Coverage**: Every Jenkins stage has a corresponding GHA equivalent with proof of functional parity
- **R2 Secrets & Auth**: All credentials migrated, scoped correctly, OIDC where specified, no plaintext exposure
- **R3 CI/CD Practices**: Workflows follow all best practices specified in the migration spec
- **R4 Proof Quality**: Proof artifacts contain meaningful evidence of parity, not just "workflow ran"
- **R5 Git Traceability**: Commits map to migration tasks with clear progression
- **R6 Cutover Readiness**: Parallel run data exists, rollback documented, communication plan in place

## Validation Process

### Step 1 — Input Discovery

- Execute Auto-Discovery Protocol to locate Migration Spec + Discovery Report + Task List
- Use `git log --stat -10` to identify migration implementation commits
- Parse proof artifacts from `./docs/specs/[NN]-migration-[pipeline-name]/[NN]-proofs/`
- Read the migration spec's platform delta analysis, secrets strategy, and cutover criteria

### Step 2 — Parity Analysis

For every Jenkins stage documented in the migration spec:

1. Identify the corresponding GHA job/step
2. Locate the proof artifact demonstrating parity
3. Verify the proof shows equivalent behavior (same artifacts, same test results, same deploy targets)
4. Mark as **Verified**, **Failed**, or **Unknown**

### Step 3 — Secrets Audit

For every credential in the migration spec's secrets strategy:

1. Verify the GHA secret exists (referenced in workflow YAML)
2. Verify proper scoping (repo/org/environment matches spec)
3. Verify OIDC is used where the spec requires it
4. Verify no plaintext secrets in:
   - Workflow YAML files
   - Proof artifact files
   - Composite action or reusable workflow files
   - Shell scripts committed to the repository
5. Verify `permissions:` blocks include `id-token: write` where OIDC is used

### Step 4 — Best Practices Audit

For every `.github/workflows/*.yml` file:

1. `permissions:` block present with explicit, minimum scopes
2. All `uses:` directives pinned to full commit SHA (40-char hex)
3. `concurrency:` group configured with appropriate `cancel-in-progress` setting
4. `timeout-minutes:` set on every job
5. `actions/cache` used for dependency management where applicable
6. Environment protection rules configured for deployment jobs
7. No `pull_request_target` with fork code checkout in privileged context

### Step 5 — Coverage Verification

Cross-reference:

1. **Jenkins stages** → at least one GHA job/step per stage
2. **Plugin dependencies** → all have GHA equivalents implemented
3. **Platform deltas** → all are resolved (check the spec's completeness checklist)
4. **Shared libraries** → all functions have composite action or reusable workflow replacements
5. **Integration points** → all external connections are functional
6. **Notification channels** → all deliver equivalent messages

### Step 6 — Cutover Readiness

Evaluate:

1. **Parallel run data**: Do proof artifacts show side-by-side Jenkins and GHA results?
2. **Rollback procedure**: Is it documented? Can Jenkins be re-enabled quickly?
3. **Communication**: Is there a plan for notifying teams about the cutover?
4. **Monitoring**: Are GHA workflow run notifications configured for the transition period?

## Output (single Markdown report)

### 1) Executive Summary

- **Overall**: PASS / FAIL (list gates tripped)
- **Cutover Ready**: Yes / No with one-sentence rationale
- **Key Metrics**: % Stages Migrated, % Secrets Verified, % Best Practices Compliant

### 2) Parity Matrix (required)

#### Stage Parity

| Jenkins Stage | GHA Equivalent | Parity Status | Evidence |
|---|---|---|---|
| [stage] | [job/step] | Verified / Failed / Unknown | [proof artifact reference] |

#### Secrets Parity

| Credential | Jenkins Type | GHA Configuration | Status | Evidence |
|---|---|---|---|---|
| [id] | [type] | [scope + method] | Verified / Failed | [proof reference] |

#### Integration Parity

| Integration | Jenkins Method | GHA Method | Status | Evidence |
|---|---|---|---|---|
| [service] | [plugin/step] | [action/step] | Verified / Failed / Unknown | [proof reference] |

### 3) Best Practices Audit

| Workflow File | Permissions | Action Pinning | Concurrency | Timeout | Cache | Status |
|---|---|---|---|---|---|---|
| [file] | ✅/❌ | ✅/❌ | ✅/❌ | ✅/❌ | ✅/❌/N/A | Pass/Fail |

### 4) Cutover Readiness Checklist

- [ ] All pipeline stages have GHA equivalents (Gate E)
- [ ] All secrets migrated and verified (Gate B)
- [ ] All best practices enforced (Gate C)
- [ ] Parallel run completed with no parity failures (Gate A)
- [ ] Rollback procedure documented and tested (Gate D)
- [ ] No security vulnerabilities introduced (Gate F)
- [ ] Team notified of cutover timeline
- [ ] Jenkins pipeline preserved for rollback (not yet disabled)

### 5) Validation Issues

For each issue:

| Severity | Issue | Impact | Recommendation |
|---|---|---|---|
| CRITICAL/HIGH/MEDIUM/LOW | [description with evidence] | [what breaks] | [how to fix] |

### 6) Evidence Appendix

- Git commits analyzed with file changes
- Proof artifact summaries
- actionlint results
- Parallel run comparison data (if available)

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
- No parallel run evidence before cutover recommendation
- Proof artifacts missing for parent tasks
- OIDC not used where spec requires it
- `pull_request_target` with fork checkout in privileged context

## What Comes Next

Once validation passes all gates:

- **All PASS**: "Migration validated. Proceed with cutover per the strategy documented in the migration spec. Re-enable Jenkins pipeline as rollback path until the monitoring period expires."
- **Any FAIL**: "Migration has [N] blocking issues. Resolve the issues listed above, re-run affected tasks from `/SDM-4-execute-migration`, and re-validate with `/SDM-5-validate-migration`."

---

**Validation Completed:** [Date+Time]
**Validation Performed By:** [AI Model]
