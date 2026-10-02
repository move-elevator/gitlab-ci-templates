# AGENTS.md

## Project overview

Reusable GitLab CI/CD job templates for [move:elevator](https://www.move-elevator.de/) PHP and Node.js projects, primarily TYPO3. The repository contains only YAML. Consuming projects include the templates through raw GitHub URLs in their `.gitlab-ci.yml` and override variables:

```yaml
include:
  - 'https://raw.githubusercontent.com/move-elevator/gitlab-ci-templates/main/.base.yaml'
  - 'https://raw.githubusercontent.com/move-elevator/gitlab-ci-templates/main/build/build-php.yaml'
```

`.gitlab-ci.yml.dist` is a complete example configuration for a consuming project.

## Structure

- `.base.yaml`: foundation. Default Docker image (`ghcr.io/move-elevator/php8.4-composer:latest`), pipeline variables, stages, and reusable anchors (`.ssh`, `.composer-auth`, `.check-deployment-dependencies`, `.normalize-feature-name`, `.export-environment-url`)
- `build/`: composer install, npm build, release notes
- `analyze/`: static analysis and linting (PHPStan, Rector, PHP CS Fixer, ESLint, Stylelint, yamllint, xmllint, editorconfig, Fluid, TypoScript, composer, content blocks)
- `deploy/`: feature, stage and prod deployments with rollback and cleanup, using [deployer](https://deployer.org/) and [deployer-tools](https://github.com/move-elevator/deployer-tools)
- `sync/`: database and file sync to feature environments
- `cache/`: cache warmup after deploy
- `test/`: post-deployment acceptance tests (Codeception, HTTP client)
- `release/`: release preparation and GitLab release creation on version tags
- `security/`: scheduled dependency audits and SBOM
- `.github/workflows/cgl.yml`: CI for this repository
- `CODEOWNERS`

Stage order, defined in `.base.yaml`: `build`, `analyze`, `deploy.feature`, `deploy.stage`, `sync`, `cache`, `test`, `deploy.prod`, `cache.prod`, `test.prod`, `release`, `security`.

## Conventions

- One YAML file per CI job, named `{stage}-{environment}-{tool}.yaml`. Job names are colon-separated, for example `deploy:feature:cleanup`
- Reusable configuration lives in `.base.yaml` as anchors and is used via `extends` or `!reference`
- Most jobs share one rule pattern: `$CI_PIPELINE_SOURCE == "web"` always runs, tags, schedules, downstream pipelines and merge requests are skipped (except release, security and cleanup jobs), then branch and file-change conditions apply, and the default is to run
- Feature branch regex: `^(main|feature-.*|[A-Z]{2,}-\d+-.*|[^/]+/.*)$`. Any `prefix/rest` branch matches too, deployer-tools normalizes the slash to a hyphen for the instance name
- Scheduled jobs require the `SCHEDULE_TASK_NAME` variable to match the job name
- Feature environments auto-stop after 3 months. `resource_group` prevents concurrent deployments per branch
- Variables set by consuming projects include `SSH_KEY`, `SSH_USER_STAGE`, `SSH_HOST_STAGE`, `SSH_USER_PROD`, `SSH_HOST_PROD`, `SSH_PORT`, `DOMAIN_STAGE`, `DOMAIN_PROD`, `BUILD_COMPOSER_VERSION`, `BUILD_NODE_VERSION`

Job dependencies:

```
build:php -> analyze:php:*, deploy:*, sync:*, security:*
build:node -> analyze:js:lint, analyze:style:lint
deploy:feature -> cache:feature:warmup -> test:feature:*
deploy:prod -> cache:prod:warmup -> test:prod:*
build:release-notes -> release (tags only)
```

## Development commands

There is no build step.

## Testing

No automated tests are configured. Validate changes by including the template in a GitLab pipeline.

## Code style and linting

```bash
yamllint -d relaxed .
```

CI (`.github/workflows/cgl.yml`) runs exactly this on every push that touches `*.yaml` or `*.yml`. No other linter is configured.

## Git workflow

- Commit format: `<type>: <description>` with `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`
- No co-author trailers
