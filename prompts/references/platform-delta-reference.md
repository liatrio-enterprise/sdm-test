# Jenkins-to-GitHub Actions Platform Delta Reference

Use this table when mapping Jenkins concepts to GitHub Actions equivalents. This is the authoritative source for platform differences across all SDM prompts.

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
| `environment {}` block (stage) | `jobs.<id>.env:` or top-level `env:` | Env vars used by only one job belong at job level; env vars shared across multiple jobs belong at workflow level |
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
| `emailext` | `dawidd6/action-send-mail` | Third-party action; no native email |
| `slackSend` | `slackapi/slack-github-action` | Official Slack action available |
| `withSonarQubeEnv` | `sonarsource/sonarqube-scan-action` | Official SonarQube action |
| `cleanWs()` / `deleteDir()` | Not needed — runners are ephemeral | Workspace is automatically cleaned |
| `buildDiscarder` / `logRotator` | Workflow run retention settings | Configure in repo settings or via API |
| `Jenkinsfile` in repo root | `.github/workflows/*.yml` | Multiple workflow files; naming convention matters |
| `tool 'JDK-17'` / `jdk` agent option | `actions/setup-java@v4` with `distribution` + `java-version` | Specify distribution (e.g., `temurin`); supports caching via `cache: maven` or `cache: gradle` |
| `mvn` / `./mvnw` in `sh` steps | `actions/setup-java` + `run: ./mvnw --batch-mode verify` | Use `--batch-mode --update-snapshots` for CI; prefer `mvnw` for version consistency |
| `./gradlew` in `sh` steps | `gradle/actions/setup-gradle` + `run: ./gradlew build` | Action manages caching and daemon lifecycle; pin to SHA |
| Maven `settings.xml` via `configFileProvider` | `actions/setup-java` `server-id` / `server-username` / `server-password` | Auto-generates `~/.m2/settings.xml`; no config file plugin needed |
| `docker.build()` / `docker.push()` | `docker/build-push-action` + `docker/login-action` | Or use Spring Boot buildpacks: `mvn spring-boot:build-image` / `./gradlew bootBuildImage` |
| `junit '**/target/surefire-reports/*.xml'` | `dorny/test-reporter` or `mikepenz/action-junit-report` | Publishes test results as PR check annotations |
