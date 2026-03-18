---
name: SDM-5-validate-migration
description: "Validate migration completeness with parity testing, best practices audit, and post-migration issue review"
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

You have completed the **migration execution** phase and are now entering **validation**. This is where you verify that the GitHub Actions workflows are functionally equivalent to the Jenkins pipelines, all best practices are followed, and the post-migration GitHub issues capture all deferred items.

### Workflow Integration

This validation phase serves as the **quality gate** for the migration:

**Value Chain Flow:**

- **Implementation → Validation**: Transforms working GHA workflows into verified migration
- **Validation → Report**: Creates evidence of migration parity and verifies post-migration issues are filed

**Critical Dependencies:**

- **Migration Spec** serves as the acceptance criteria for functional parity
- **Proof Artifacts** from SDM-4 provide the evidence base for validation
- **Platform Delta Analysis** defines what "equivalent" means for each Jenkins concept
- **Post-Migration GitHub Issues** capture all deferred items for follow-up

**What Breaks the Chain:**

- Missing proof artifacts → parity cannot be verified
- Incomplete platform delta coverage → migrated behaviors may be missing
- Missing post-migration GitHub issues → deferred items lost

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

## Evaluation Rubric (score each 0–3)

Map score to severity: 0→CRITICAL, 1→HIGH, 2→MEDIUM, 3→OK.

- **R1 Parity Coverage**: Every Jenkins stage has a corresponding GHA equivalent with proof of functional parity
- **R2 CI/CD Practices**: Workflows follow all best practices specified in the migration spec
- **R3 Proof Quality**: Proof artifacts contain meaningful evidence of parity, not just "workflow ran"
- **R4 Git Traceability**: Commits map to migration tasks with clear progression
- **R5 Post-Migration Issues**: Post-migration GitHub issues are comprehensive and actionable

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

### Step 3 — Best Practices Audit

For every `.github/workflows/*.yml` file:

1. `permissions:` block present with explicit, minimum scopes
2. All `uses:` directives pinned to full commit SHA (40-char hex)
3. `concurrency:` group configured with appropriate `cancel-in-progress` setting
4. `timeout-minutes:` set on every job
5. `actions/cache` used for dependency management where applicable
6. No `pull_request_target` with fork code checkout in privileged context
7. No secrets hardcoded in YAML or logs
8. Application workflows live in the application repo's `.github/workflows/`; reusable workflows live in the repository specified by the migration spec's Output Strategy

### Step 4 — Coverage Verification

Cross-reference:

1. **Jenkins stages** → at least one GHA job/step per stage
2. **Plugin dependencies** → all have GHA equivalents implemented
3. **Platform deltas** → all are resolved (check the spec's completeness checklist)
4. **Shared libraries** → all functions have reusable workflow or inline replacements, and reusable workflows are in the location specified by the migration spec's Output Strategy

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
