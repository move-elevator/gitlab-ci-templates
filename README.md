# GitLab-CI Templates

This repository provides useful templates for reusable GitLab-CI jobs.

## Installation

See [.gitlab-ci.yml.dist](.gitlab-ci.yml.dist) for an example GitLab-CI configuration.

Use `include` to reference template files:

```yaml
include:
   - 'https://raw.githubusercontent.com/move-elevator/gitlab-ci-templates/main/.base.yml'
   - 'https://raw.githubusercontent.com/move-elevator/gitlab-ci-templates/main/build/build-php.yml'
   - ... 
```

Extend and override configuration variables:

```yaml
variables:
  BUILD_NODE_VERSION: "22"
```

Extend and override further ci jobs.