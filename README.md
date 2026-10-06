<div align="center">

# GitLab-CI Templates

[![CGL](https://img.shields.io/github/actions/workflow/status/move-elevator/gitlab-ci-templates/cgl.yml?label=cgl&logo=github)](https://github.com/move-elevator/gitlab-ci-templates/actions/workflows/cgl.yml)
[![License](https://img.shields.io/github/license/move-elevator/gitlab-ci-templates)](LICENSE.md)

</div>

This repository provides useful templates for reusable GitLab-CI jobs in [move:elevator](https://www.move-elevator.de/) projects. It is not meant to be used anywhere else.

## 🔥 Installation

See [.gitlab-ci.yml.dist](.gitlab-ci.yml.dist) for an example GitLab-CI configuration.

Use `include` to reference template files:

```yaml
include:
   - 'https://raw.githubusercontent.com/move-elevator/gitlab-ci-templates/main/.base.yml'
   - 'https://raw.githubusercontent.com/move-elevator/gitlab-ci-templates/main/build/build-php.yml'
   - ...
```

Extend and override configuration variables, see [.base.yaml](.base.yaml) for predefined variables.

```yaml
variables:
  BUILD_NODE_VERSION: "22"
```

Extend and override further ci jobs.

## 📂 Templates

- `.base.yml`: Base configuration, including common variables and stages, required for all projects.

### Build

Build the project and its assets.

Includes:
- `build/build-php.yml`
- `build/build-node.yaml`

### Analysis

Analyze code quality and static analysis.

Includes:
- `analyze/analyze-composer-lint.yaml`
- `analyze/analyze-content-blocks-lint.yaml`
- `analyze/analyze-editorconfig.yaml`
- `analyze/analyze-fluid-lint.yaml`
- `analyze/analyze-js-lint.yaml`
- `analyze/analyze-php-cs-fixer.yaml`
- `analyze/analyze-php-rector.yaml`
- `analyze/analyze-php-stan.yaml`
- `analyze/analyze-style-lint.yaml`
- `analyze/analyze-typescript-lint.yaml`
- `analyze/analyze-typoscript-lint.yaml`
- `analyze/analyze-xml-lint.yaml`
- `analyze/analyze-yaml-lint.yaml`

> [!NOTE]
> The `analyze` jobs have changes triggers for the respective files, so they only run when relevant files are changed.

### Feature Branch Deployment

Deploy feature branches to a dedicated environment.

The deployment uses [deployer](https://deployer.org/) and [deployer-tools](https://github.com/move-elevator/deployer-tools) as deployment base.

Includes:
- `deploy/deploy-feature.yaml`
- `deploy/deploy-feature-rollback.yaml`
- `deploy/deploy-feature-cleanup.yaml`
- `deploy/deploy-feature-cleanup-downstream.yaml`

> [!NOTE]
> The cleanup of a feature branch is a little bit tricky. It may happen that the branch has been deleted, triggering the `deploy:feature:cleanup` job. However, since the application code in the branch is no longer available at this point and the cleanup logic and configuration are therefore no longer present, the cleanup is delegated to another branch (usually the `main` branch) via the downstream pipeline.

### Prod Deployment

Deployment to production environment.

The deployment uses [deployer](https://deployer.org/) and [deployer-tools](https://github.com/move-elevator/deployer-tools) as deployment base.

Includes:
- `deploy/deploy-prod.yaml`
- `deploy/deploy-prod-rollback.yaml`

> [!IMPORTANT]
> Production deployments are restricted to semver-tagged versions only (e.g. `v1.2.3` or `1.2.3`).

### Stage Deployment

Deployment to staging environment.

The deployment uses [deployer](https://deployer.org/) and [deployer-tools](https://github.com/move-elevator/deployer-tools) as deployment base.

Includes:
- `deploy/deploy-stage.yaml`

> [!NOTE]
> Use this template if the project wants a simple stage deployment and doesn't need an advanced feature branch deployment.

### Release

Create a GitLab release with release notes.

Includes:
- `build/build-release-notes.yaml`
- `release/release.yaml`

The release notes come from one of two mutually exclusive templates — include exactly one:

- `build/build-release-notes.yaml`: fetches them from GitLab's changelog API.
- `build/build-release-notes-changelog.yaml`: extracts the section for the current tag from
  `CHANGELOG.md`. Use this one for projects whose changelog is generated from conventional
  commits, which the GitLab API does not understand.

The asset link on the release page defaults to the project's container image and is overridden
with `RELEASE_ASSET_LINK_NAME` and `RELEASE_ASSET_LINK_URL`.

#### Automated releases

Includes:
- `release/release-prepare.yaml`
- `build/build-release-notes-changelog.yaml`
- `release/release.yaml`

`release:prepare` is a manual job on the default branch that bumps the version, generates
`CHANGELOG.md`, folds both into one `chore(release): version X.Y.Z` commit authored by whoever
pressed the button, and pushes the commit (with `ci.skip`) and the tag. The tag pipeline then
builds the release notes, publishes the GitLab Release and deploys to production.

It needs [version-bumper](https://github.com/eliashaeussler/version-bumper) with a
`version-bumper.yaml`, an npm script `changelog`, and a masked, protected `RELEASE_TOKEN`
variable holding a token with `write_repository` scope.

> [!IMPORTANT]
> Only users who may push to a protected branch can run manual jobs on it. Protect the default
> branch and set "Allowed to push and merge" to the people who may release — that, and not the
> job definition, is what restricts who can cut a release.

> [!NOTE]
> The job ends green without releasing when the branch tip is already the last tag — there is
> nothing to put in the changelog. Set `RELEASE_RANGE` to `major`, `minor` or `patch` to release
> anyway.

> [!NOTE]
> Set `RELEASE_DRY_RUN` to any value when starting the job to rehearse a release: everything
> runs, but both pushes become `git push --dry-run`. With it set the job is also offered on
> non-default branches.

### Testing

Run acceptance tests using [Codeception](https://codeception.com/).

Includes:
- `test/test-feature-codeception.yaml`
- `test/test-prod-codeception.yaml`

#### Ghost Inspector

Run the tests of [Ghost Inspector](https://ghostinspector.com/) suites against the feature branch instance with [`@move-elevator/recorded-test-runner`](https://www.npmjs.com/package/@move-elevator/recorded-test-runner), so they run in every branch pipeline without consuming Ghost Inspector test runs. The tests stay maintained in Ghost Inspector and are exported fresh in every pipeline.

Includes:
- `build/build-ghost-inspector.yaml`
- `test/test-feature-ghost-inspector.yaml`

| Variable | Purpose |
|---|---|
| `GI_API_KEY` | API key, masked but not protected, because feature branches need it. Use the key of a dedicated Ghost Inspector user, since it can also start test runs |
| `GI_SUITE_IDS` or `GI_FOLDER_ID` | Comma-separated suite IDs, or a folder whose suites are all exported |
| `GI_HTTP_AUTH_USER`, `GI_HTTP_AUTH_PASSWORD` | Optional basic auth for the feature instance. Credentials stored in Ghost Inspector are never exported |

`build:ghost-inspector` exports the suites and every test they import via "execute", including shared modules from other suites. `test:feature:ghost-inspector` runs them in every viewport configured in the suite against `https://$DOMAIN_STAGE/<feature>/`, with URLs of the recorded site rewritten to that instance. The JUnit report shows up in the merge request, the JSON report and screenshots of failed runs are kept as artifacts. Supported step commands, withheld values and known differences to Ghost Inspector are documented with the runner.

> [!WARNING]
> Because `GI_API_KEY` is not protected, anyone who can push a branch can read it through a pipeline, as with every unprotected variable. Masking does not prevent that. Use a dedicated Ghost Inspector user, so the key can be revoked on its own, and do not use these templates in projects with untrusted contributors.

> [!NOTE]
> Both jobs are `allow_failure: true`. An unreachable Ghost Inspector API must not block the pipeline, and tests written for prod can fail on feature instances for reasons unrelated to the change: selectors with absolute paths such as `a[href="/page"]` (use `a[href$="/page"]`), links hardcoded to `/`, or assertions on live data.

> [!IMPORTANT]
> `GI_RUNNER_VERSION` and `GI_PLAYWRIGHT_VERSION` in `.base.yaml` are bumped together: the browser image has to match the Playwright version of that runner release. The export job needs Node.js 22.13 or later, so `BUILD_NODE_VERSION` must be at least `22`.

### Cache Warmup

Warm up the cache after deployment using [EXT:typo3-warming](https://github.com/eliashaeussler/typo3-warming).

Includes:
- `cache/cache-feature-warmup.yaml`
- `cache/cache-prod-warmup.yaml`

### Security

Run security checks using `audit` tools.

Includes:
- `security/security-composer-check-scheduled.yaml`
- `security/security-npm-check-scheduled.yaml`

> [!NOTE]
> The `*:scheduled` jobs above need to be scheduled in GitLab-CI.

> [!IMPORTANT]
> A pipeline schedule needs to have set the variable `SCHEDULE_TASK_NAME` to the desired task, e.g. `security:composer:check:scheduled` or `security:npm:check:scheduled`.
> This is a workaround (see according rules) to address specific jobs in a scheduled pipeline.

Generate a CycloneDX SBOM (Software Bill of Materials) of all locked composer and npm dependencies using [syft](https://github.com/anchore/syft).
The SBOM is stored as job artifact and published to the project's Generic Package Registry, which serves as technical documentation in terms of the Cyber Resilience Act (CRA).

Includes:
- `security/security-sbom.yaml`

Runs on semantic version tags only, matching the prod deployment. Published as `packages/generic/sbom/<version>/sbom.cdx.json`, where `<version>` is the tag without a leading `v`. Dev dependencies are excluded, so the SBOM describes the shipped application rather than the build toolchain.

> [!IMPORTANT]
> Technical documentation has to be kept for the declared support period. Make sure the project has no cleanup policy that removes generic packages.

## ⭐ License

This project is licensed under [GNU General Public License 3.0 (or later)](LICENSE).