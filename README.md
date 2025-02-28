# Skip Continuous Integration Workflows

This repository host shared workflows that can be used
by Skip framework and app projects to build, test, and
deploy standard Skip projects.

## Frameworks

For building and testing a framework,
create a `.github/workflows/ci.yml` file like this:

```yaml
name: lib-name
on:
  push:
    branches: '*'
    tags: "[0-9]+.[0-9]+.[0-9]+"
  workflow_dispatch:
  pull_request:

permissions:
  contents: write

jobs:
  call-workflow:
    uses: skiptools/actions/.github/workflows/skip-framework.yml@v1
```

## Applications

For building and testing an app,
create a `.github/workflows/ci.yml` file like this:

```yaml
name: app-name
on:
  push:
    branches: '*'
    tags: "[0-9]+.[0-9]+.[0-9]+"
  workflow_dispatch:
  pull_request:

permissions:
  contents: write
  id-token: write
  attestations: write

jobs:
  call-workflow:
    uses: skiptools/actions/.github/workflows/skip-app.yml@v1
```

## Releasing

To release a version and update the symbolic "v1" tag, run:

```
git tag v1.0.0 && git push --tags && git tag -fa v1 -m "Update v1 tag" && git push origin v1 --force
```

