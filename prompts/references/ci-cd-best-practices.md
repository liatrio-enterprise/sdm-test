# CI/CD Best Practices (Enforced Requirements)

These best practices are the baseline quality bar for migrated workflows. They apply across SDM-2 (spec), SDM-4 (execution), and SDM-5 (validation).

## Security

- **Permissions block**: Every workflow MUST include `permissions:` with minimum required scopes. Default to `contents: read` and only add write permissions where explicitly needed
- **Action pinning**: All third-party actions MUST be pinned to full commit SHA, not tags or branches (e.g., `uses: actions/checkout@abc123def456` not `@v4`)
- **OIDC over stored credentials**: For cloud provider access (AWS, Azure, GCP), prefer OIDC federation over long-lived access keys. Use `aws-actions/configure-aws-credentials` with `role-to-assume` and OIDC
- **Fork PR security**: Never use `pull_request_target` with checkout of fork code in a privileged context
- **Log masking**: Use `::add-mask::` for any dynamically generated sensitive values
- **GITHUB_TOKEN scoping**: Prefer `GITHUB_TOKEN` over PATs; scope to minimum permissions

## Efficiency

- **Environment variable scoping**: Hoist environment variables to the narrowest scope that covers all usage. If an env var is used by a single job, define it under that job's `env:` key. If the same env var appears in multiple jobs, promote it to the workflow-level `env:` block. Never duplicate the same env var across multiple jobs when a workflow-level declaration suffices
- **Caching**: Use `actions/cache` for package manager dependencies (npm, pip, maven, gradle). Define cache keys based on lockfile hashes
- **Concurrency groups**: Use `concurrency:` to prevent duplicate workflow runs. Set `cancel-in-progress: true` for PR workflows
- **Matrix builds**: Use `strategy.matrix` for multi-version or multi-platform testing instead of duplicating jobs
- **YAML anchors and aliases**: Use YAML anchors (`&`) to define reusable content and aliases (`*`) to reference it elsewhere in the workflow. Use anchors to share environment variable blocks across jobs (`env: &env_vars` / `env: *env_vars`) and to reuse entire job configurations (`&base_job` / `*base_job`). This reduces duplication and keeps workflows maintainable
- **Reusable workflows**: Extract shared logic into reusable workflows (`.github/workflows/reusable-*.yml`) called with `workflow_call`
- **Composite actions**: Create repo-local composite actions in `.github/actions/[name]/action.yml` for repeated step sequences

## Reliability

- **Environment protection rules**: Configure required reviewers and deployment branches for production environments
- **Branch protection**: Require status checks to pass before merging; protect workflow files from unauthorized changes
- **Immutable artifacts**: Upload build artifacts with `actions/upload-artifact@v4`; use content-addressable names where possible
- **Timeout configuration**: Set `timeout-minutes` on every job to prevent runaway builds

## Spring / Java Application Patterns

Apply when the pipeline builds a Java/Spring project:

- **JDK setup**: Use `actions/setup-java@v4` with explicit `java-version`, `distribution` (prefer `temurin`), and `cache` parameter. Set `cache: maven` for Maven projects or `cache: gradle` for Gradle projects — this replaces manual `actions/cache` configuration for dependencies
- **Maven builds**: Run with `--batch-mode --update-snapshots` flags in CI. `--batch-mode` suppresses interactive prompts and produces cleaner logs; `--update-snapshots` ensures SNAPSHOT dependencies are current
- **Gradle builds**: Use the `gradle/actions/setup-gradle` action (SHA-pinned) instead of invoking `./gradlew` directly — it provides caching, build scan integration, and daemon management
- **Maven wrapper**: If the project includes `mvnw`, use `./mvnw` instead of `mvn` to ensure build reproducibility
- **Multi-JDK testing**: Use `strategy.matrix` with `java-version: ['17', '21']` to test across JDK versions in parallel
- **Artifact uploads**: Maven outputs to `target/`, Gradle to `build/libs/`. Upload JARs/WARs with `actions/upload-artifact@v4` using content-addressable names (e.g., include `${{ github.sha }}`)
- **Container images**: For Spring Boot apps, prefer Spring Boot's built-in buildpack support (`mvn spring-boot:build-image` / `./gradlew bootBuildImage`) over manual Dockerfiles when possible. When a Dockerfile is needed, use `docker/build-push-action` with `docker/login-action`
- **Container registry**: Use `ghcr.io` with `GITHUB_TOKEN` for GitHub Container Registry (requires `packages: write` permission), or configure external registries via their respective login actions
- **Package publishing**: For Maven Central, use `actions/setup-java@v4` with `server-id`, `server-username`, and `server-password` to configure `~/.m2/settings.xml` automatically. For GitHub Packages, use `GITHUB_TOKEN` with `packages: write`
- **Test reporting**: Use `dorny/test-reporter` or `mikepenz/action-junit-report` (SHA-pinned) to publish JUnit XML results as PR check annotations
- **Build attestation**: For container images, use `actions/attest-build-provenance` to generate SLSA provenance attestations

## Verification Checklist

Use this checklist during execution (SDM-4) and validation (SDM-5):

- [ ] `permissions:` block present with minimum required scopes
- [ ] All third-party actions pinned to full commit SHA
- [ ] `concurrency:` group configured (for PR-triggered workflows)
- [ ] `timeout-minutes:` set on every job
- [ ] No secrets hardcoded in YAML
- [ ] Environment variables scoped correctly (job-level vs workflow-level)
- [ ] YAML anchors and aliases used to eliminate duplication where applicable
- [ ] `actions/cache` used for dependency caching where applicable
- [ ] Environment protection rules configured for deployment jobs

**Spring/Java additions:**
- [ ] `actions/setup-java@v4` with explicit `distribution`, `java-version`, and `cache`
- [ ] Maven builds use `--batch-mode --update-snapshots`
- [ ] Gradle builds use `gradle/actions/setup-gradle` (SHA-pinned)
- [ ] Maven wrapper (`./mvnw`) used when present
- [ ] Container images use buildpacks or `docker/build-push-action`
- [ ] Test results published via reporter action (SHA-pinned)
