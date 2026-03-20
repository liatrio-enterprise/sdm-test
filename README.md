<div align="center">
    <h1>Spec-Driven Migration (SDM) Workflow</h1>
    <h3><em>Migrate Jenkins pipelines to GitHub Actions with a repeatable, AI-guided workflow.</em></h3>
</div>

<p align="center">
    <strong>Structured prompts for collaborating with AI agents to deliver reliable Jenkins-to-GitHub Actions migrations.</strong>
</p>

## Overview

This repository provides **structured prompts** (Markdown files) that guide AI assistants through a complete Jenkins-to-GitHub Actions migration workflow:

- **Discover**: audit existing Jenkins pipelines, plugins, credentials, shared libraries, and infrastructure
- **Specify**: generate a migration spec with platform delta analysis, secrets strategy, and CI/CD best practices
- **Plan**: break migration work into ordered, implementable tasks using infrastructure-first sequencing
- **Execute**: implement GitHub Actions workflows with checkpoints, actionlint validation, and secret verification gates
- **Validate**: verify migration completeness with parity testing, best practices audit, and post-migration issue review

Think of these prompts as reusable playbooks that keep AI agents focused and consistent across the entire migration lifecycle.

## Table of Contents

- [TLDR / Quickstart](#tldr--quickstart)
- [Details for the 5-step workflow](#details-for-the-5-step-workflow)
- [Artifacts and directory layout](#artifacts-and-directory-layout)
- [Context verification markers](#context-verification-markers)
- [Security best practices](#security-best-practices)
- [Contributing](#contributing)
- [License](#license)

## TLDR / Quickstart

### Installation options

#### Option A: Install as Slash Commands (Recommended)

Install these prompts as native `/slash-commands` in your AI assistant (Cursor, Windsurf, Claude Code, etc.) using the [slash-command-manager](https://github.com/liatrio-labs/slash-command-manager) utility:

**Prerequisite:** `uvx` comes from [uv](https://docs.astral.sh/uv/). Install uv first if you don't already have it:

- (Mac): `brew install uv`
- (Windows): `winget install astral-sh.uv`

##### Install SDM w/ Bash (Mac)

```bash
uvx --from git+https://github.com/liatrio-labs/slash-command-manager \
  slash-man generate \
  --github-repo liatrio-labs/spec-driven-workflow \
  --github-branch main \
  --github-path prompts/
```

##### Install SDM w/ PowerShell (Windows)

```ps
uvx --from git+https://github.com/liatrio-labs/slash-command-manager `
  slash-man generate `
  --github-repo liatrio-labs/spec-driven-workflow `
  --github-branch main `
  --github-path prompts/
```

**What this command does:**

- `uvx` runs a Python tool without installing it globally (like `npx` for Python)
- Fetches the `slash-command-manager` tool from GitHub
- Auto-detects your installed AI assistants from the list of supported tools
- Downloads the prompt files for each supported tool from the `prompts/` directory
- Installs them as slash commands for each supported tool

**Result:** you can now type `/SDM-1-discovery-assessment` in your AI assistant to start the workflow.

**Where to use the slash commands:** in AI chat UIs (e.g., Windsurf, Claude Code) type `/` in the chat input. Some AI assistants require being in "Agent" or "Code" mode for slash commands to appear.

#### Option B: Manual Copy-Paste (No Installation)

Copy the contents of a prompt file directly from `prompts/` and paste it into your AI chat. The AI will follow the structured instructions in the prompt.

### Quick "try it" flow

1. Run `/SDM-1-discovery-assessment` and point it at your Jenkinsfile or Jenkins pipeline configuration.
2. Next, use `/SDM-2-generate-migration-spec` to create a migration spec from the discovery report.
3. Then run `/SDM-3-generate-migration-tasks` to generate an ordered task list from the spec.
4. Execute `/SDM-4-execute-migration` to implement the GitHub Actions workflows task by task.
5. Finally, apply `/SDM-5-validate-migration` to verify migration completeness and parity.

## Details for the 5-step workflow

Each step uses a different prompt file and produces specific artifacts in `docs/specs/`.

1. **Discovery and Assessment** ([`prompts/SDM-1-discovery-assessment.md`](./prompts/SDM-1-discovery-assessment.md))
   - **What it does**: audits existing Jenkins pipelines, plugins, credentials, shared libraries, and infrastructure
   - **Output**: discovery report documenting the current Jenkins environment
   - **Why**: builds the factual foundation for the entire migration — everything downstream traces back to this inventory

2. **Generate Migration Spec** ([`prompts/SDM-2-generate-migration-spec.md`](./prompts/SDM-2-generate-migration-spec.md))
   - **What it does**: transforms the discovery inventory into a migration specification with platform delta analysis, secrets strategy, and CI/CD best practices
   - **Output**: migration specification — the single source of truth for the rest of the workflow
   - **Why**: maps Jenkins concepts to GitHub Actions equivalents and identifies gaps before implementation begins

3. **Generate Migration Tasks** ([`prompts/SDM-3-generate-migration-tasks.md`](./prompts/SDM-3-generate-migration-tasks.md))
   - **What it does**: breaks the migration spec into ordered, implementable tasks using infrastructure-first sequencing (foundation → pipeline → post-migration)
   - **Output**: ordered task list with parent tasks and subtasks
   - **Why**: CI/CD migrations have real dependency chains — task ordering ensures each layer is verified before the next begins

4. **Execute Migration** ([`prompts/SDM-4-execute-migration.md`](./prompts/SDM-4-execute-migration.md))
   - **What it does**: converts Jenkins pipeline configurations into working GitHub Actions workflows with actionlint validation, secret verification gates, and proof artifacts
   - **Output**: GitHub Actions workflow files and proof artifacts for each task
   - **Why**: structured execution with verification at each step prevents cascading issues

5. **Validate Migration** ([`prompts/SDM-5-validate-migration.md`](./prompts/SDM-5-validate-migration.md))
   - **What it does**: validates migration completeness with parity testing, best practices audit, and post-migration issue review
   - **Output**: validation report with coverage matrix and any deferred items captured as GitHub issues
   - **Why**: confirms the GitHub Actions workflows are functionally equivalent to the original Jenkins pipelines

6. **SHIP IT**

## Highlights

- **Migration-focused workflow:** Purpose-built prompts for Jenkins-to-GitHub Actions migrations, covering discovery through validation.
- **Infrastructure-first sequencing:** Tasks are ordered by dependency (foundation → pipeline logic → post-migration) to prevent cascading failures.
- **Built-in CI/CD best practices:** Prompts enforce actionlint validation, secret verification gates, YAML anchors/aliases, and proper env var scoping.
- **No dependencies required:** The prompts are plain Markdown files that work with any AI assistant.
- **Context verification:** Built-in emoji markers (SDM1️⃣–SDM5️⃣) detect when AI responses follow critical instructions, helping identify context rot issues early.

## Why Spec-Driven Migration?

Jenkins-to-GitHub Actions migrations involve translating complex pipeline logic, plugin ecosystems, shared libraries, and credential management across fundamentally different platforms. Without a structured approach, migrations often miss edge cases, break CI/CD parity, or leave behind undocumented gaps.

SDM provides a lightweight, prompt-centric workflow that turns a Jenkins environment into a discovery report, a migration spec, an ordered task list, verified GitHub Actions workflows, and a validation report — all guided by AI and grounded in artifacts that humans can review at every step.

## Artifacts and directory layout

Each prompt writes Markdown outputs into `docs/specs/[NN]-spec-[feature-name]/` (where `[NN]` is a zero-padded 2-digit number: 01, 02, 03, etc.).

- **Discovery reports:** `docs/specs/[NN]-spec-[feature-name]/[NN]-discovery-[feature-name].md`
- **Migration specs:** `docs/specs/[NN]-spec-[feature-name]/[NN]-spec-[feature-name].md`
- **Task lists:** `docs/specs/[NN]-spec-[feature-name]/[NN]-tasks-[feature-name].md`
- **Proof artifacts:** `docs/specs/[NN]-spec-[feature-name]/[NN]-proofs/[NN]-task-[TT]-proofs.md`
- **Validation reports:** `docs/specs/[NN]-spec-[feature-name]/[NN]-validation-[feature-name].md`

## Context verification markers

Each prompt includes a context verification marker (SDM1️⃣ through SDM5️⃣) that appears at the start of AI responses. These markers help detect **context rot** — where AI performance degrades as input context length increases, even when tasks remain simple.

**What to expect:** Responses like `SDM1️⃣ I'll audit the Jenkinsfile...` or `SDM4️⃣ Let me implement the workflow file...`. If the marker disappears, context instructions may have been lost. For more details, see the [research documentation](docs/emoji-context-verification-research.md).

## Security Best Practices

### Protecting Sensitive Data in Proof Artifacts

Proof artifacts are committed to your repository and may be publicly visible. **Never commit real credentials or sensitive data.** Follow these guidelines:

- **Replace credentials with placeholders**: Use `[YOUR_API_KEY_HERE]`, `[REDACTED]`, or `example-key-123` instead of real API keys, tokens, or passwords
- **Use example values**: When demonstrating configuration, use dummy or example data instead of production values
- **Sanitize command output**: Review CLI output and logs for accidentally captured credentials before committing
- **Consider pre-commit hooks**: Tools like [gitleaks](https://github.com/gitleaks/gitleaks), [truffleHog](https://github.com/trufflesecurity/truffleHog), or [talisman](https://github.com/thoughtworks/talisman) can automatically scan for secrets before commits

The SDM workflow prompts include built-in reminders about security, but ultimate responsibility lies with the developer to review artifacts before committing or pushing to remotes.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). Please review [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).

## License

This project is licensed under the Apache License, Version 2.0. See the [LICENSE](LICENSE) file for details.
