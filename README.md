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
- `analyze/analyze-php-cs-fixer.yaml`
- `analyze/analyze-php-rector.yaml`
- `analyze/analyze-php-stan.yaml`
- `analyze/analyze-typoscript-lint.yaml`
- `analyze/analyze-xml-lint.yaml`
- `analyze/analyze-yaml-lint.yaml`

> [!NOTE]
> The `analyze` jobs have changes triggers for the respective files, so they only run when relevant files are changed.

### Feature Branch Deployment

Deploy feature branches to a dedicated environment.

Includes:
- `deploy/deploy-feature.yaml`
- `deploy/deploy-feature-rollback.yaml`
- `deploy/deploy-feature-stop.yaml`
- `deploy/deploy-feature-downstream.yaml`

### Prod Deployment

Deployment to production environment.

Includes:
- `deploy/deploy-prod.yaml`
- `deploy/deploy-prod-rollback.yaml`'

### Release

Create a GitLab release with release notes.

Includes:
- `build/build-release-notes.yaml`
- `release/release.yaml`'

### Testing

ToDo


## ⭐ License

This project is licensed under [GNU General Public License 3.0 (or later)](LICENSE).