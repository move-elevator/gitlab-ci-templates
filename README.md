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
- `analyze/analyze-editorconfig.yaml`
- `analyze/analyze-js-lint.yaml`
- `analyze/analyze-php-cs-fixer.yaml`
- `analyze/analyze-php-rector.yaml`
- `analyze/analyze-php-stan.yaml`
- `analyze/analyze-style-lint.yaml`
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

### Release

Create a GitLab release with release notes.

Includes:
- `build/build-release-notes.yaml`
- `release/release.yaml`

### Testing

Run acceptance tests using [Codeception](https://codeception.com/).

Includes:
- `test/test-feature-codeception.yaml`
- `test/test-prod-codeception.yaml`

### Cache Warmup

Warm up the cache after deployment using [EXT:typo3-warming](https://github.com/eliashaeussler/typo3-warming).

Includes:
- `cache/cache-feature-warmup.yaml`
- `cache/cache-prod-warmup.yaml`

## ⭐ License

This project is licensed under [GNU General Public License 3.0 (or later)](LICENSE).