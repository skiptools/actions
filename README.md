# Skip Continuous Integration Workflows

This repository host shared workflows that can be used
by Skip framework and app projects to build, test, and
deploy standard Skip projects.

## Skip Setup

You can setup Skip as part of your own workflow with:

```yaml
name: 'Skip Actions CI'
on:
  push:
    branches: [ main ]
  pull_request:
jobs:
  skip-checkup:
    runs-on: 'macos-latest'
    steps:
      - uses: skiptools/actions/setup-skip@v1
      - run: skip checkup
```

### setup-skip configuration options

The `skiptools/setup-skip` action accepts the following inputs:

```yml
  skip-version:
    description: 'The version of the Skip toolchain to install'
    required: true
    default: 'latest'
  run-doctor:
    description: 'Whether to run skip doctor'
    required: false
    default: 'true'
  verify-project:
    description: 'Path to the project to verify'
    required: false
    default: ''
  swift-version:
    description: 'Version of Swift to install'
    required: false
    default: ''
  gradle-version:
    description: 'Version of Gradle to setup'
    required: false
    default: 'current'
  install-swift-android-sdk:
    description: 'Whether to install the native Swift SDK for Android'
    required: false
    default: 'false'
  swift-android-sdk-version:
    description: 'Version of the Swift SDK for Android to install'
    required: false
    default: ''
```

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

