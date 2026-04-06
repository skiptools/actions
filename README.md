# Skip Continuous Integration Workflows

This repository hosts shared GitHub Actions and reusable workflows for building, testing, and deploying [Skip](https://skip.tools) framework and app projects.

It provides three components:

| Component | Type | Path |
|-----------|------|------|
| [setup-skip](#setup-skip-action) | Composite action | `setup-skip/action.yml` |
| [skip-framework.yml](#skip-framework-workflow) | Reusable workflow | `.github/workflows/skip-framework.yml` |
| [skip-app.yml](#skip-app-workflow) | Reusable workflow | `.github/workflows/skip-app.yml` |

---

## setup-skip Action

Installs and configures a complete Skip development environment on a GitHub Actions runner. This includes Homebrew, the Skip CLI, Gradle, and optionally Swift toolchains and the Swift SDK for Android (required for Skip Fuse projects).

Use this action when you need to set up the Skip toolchain as a step in your own custom workflow.

### Usage

```yaml
steps:
  - uses: skiptools/actions/setup-skip@v1
  - run: skip checkup
```

### Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `skip-version` | Version of the Skip toolchain to install | `'latest'` |
| `skip-source` | Homebrew source to install Skip from ([`formula`](https://formulae.brew.sh/formula/skip) or [`cask`](https://github.com/skiptools/homebrew-skip/blob/main/Casks/skip.rb)) | `'formula'` |
| `run-doctor` | Run `skip doctor` after setup to verify the environment | `'true'` |
| `verify-project` | Path to a project directory to verify with `skip verify` | `''` (disabled) |
| `swift-version` | Swift toolchain version to install via `swiftly` | `''` (use pre-installed) |
| `gradle-version` | Gradle version to set up. Set to `'none'` to skip Gradle installation | `'current'` |
| `install-swift-android-sdk` | Install the native Swift SDK for Android (required for Skip Fuse) | `'false'` |
| `swift-android-sdk-version` | Specific version of the Swift Android SDK to install | `''` (latest) |
| `swift-android-ndk-version` | Specific version of the Android NDK to install with the Swift SDK | `''` (default) |
| `verbose` | Enable verbose output for setup commands | `''` (disabled) |

### What It Does

1. Installs Homebrew (via `Homebrew/actions/setup-homebrew`)
2. Installs Gradle (via `gradle/actions/setup-gradle`) unless `gradle-version` is `'none'`
3. Installs the Skip CLI from Homebrew (formula or cask)
4. On macOS, configures Xcode to allow Skip package plugins without prompts
5. Initializes `swiftly` and optionally installs a specific Swift toolchain
6. Optionally installs the Swift SDK for Android via `skip android sdk install`
7. Optionally runs `skip doctor` and `skip verify`

### Example: Custom Workflow with Swift Android SDK

```yaml
steps:
  - uses: skiptools/actions/setup-skip@v1
    with:
      install-swift-android-sdk: 'true'
      verbose: 'true'
  - run: skip test --verbose
```

---

## skip-framework Workflow

A reusable workflow for building and testing Skip framework libraries. It performs license header verification, runs Swift and Kotlin/Robolectric tests, optionally runs instrumented Android emulator tests, builds a `skip export`, and creates GitHub releases on semver tags.

Use this workflow for any Skip framework or library project (e.g., `skip-foundation`, `skip-model`, `skip-ui`).

### Usage

Create `.github/workflows/ci.yml` in your framework repository:

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

### Inputs

| Input | Description | Type | Default |
|-------|-------------|------|---------|
| `runs-on` | JSON array of runner labels to use | `string` | `"['macos-15-intel']"` |
| `brew-install` | Additional Homebrew packages to install before building | `string` | (none) |
| `install-swift` | Swift toolchain version to install | `string` | (none) |
| `run-export` | Run `skip export` to build the project for both platforms | `boolean` | `true` |
| `run-local-tests` | Run local Swift and Robolectric tests (skipped on tag pushes) | `boolean` | `true` |
| `run-ios-tests` | Run iOS simulator tests via `xcodebuild` (skipped on tag pushes) | `boolean` | `true` |
| `run-android-tests` | Run instrumented Android emulator tests (skipped on tag pushes) | `boolean` | `true` |
| `android-native-disabled` | Skip the automatic detection of Skip Fuse (native Swift) mode | `boolean` | `false` |
| `run-android-native-build` | Force running `skip android build` for native Swift on Android | `string` | (none) |
| `emulator-api-level` | Android API level for the emulator | `string` | `'28'` |
| `emulator-channel` | Android system image channel (`stable`, `beta`, `canary`) | `string` | `'stable'` |
| `emulator-profile` | Android emulator hardware profile | `string` | `'Nexus 4'` |
| `emulator-target` | Android emulator system image target | `string` | `'default'` |
| `emulator-launcher` | Emulator launcher to use (`reactivecircus` or `skip`) | `string` | `'reactivecircus'` |

### What It Does

1. **Environment setup**: Detects whether the project uses Skip Fuse (by checking for `skip-bridge` dependency), installs the Skip toolchain and optionally the Swift Android SDK
1. **Export**: Runs `skip export` to build the project for both iOS and Android
1. **Local tests**: Runs `skip test` (or `swift test` on Linux) to execute Swift tests and Kotlin/Robolectric tests
1. **iOS simulator tests**: Runs `xcodebuild test` on an iOS simulator, automatically selecting an appropriate device for the runner OS version
1. **Android emulator tests**: Launches an Android emulator (via ReactiveCircus or `skip android emulator`) and runs `swift test --filter XCSkipTests` for instrumented tests
1. **Release**: On semver tag pushes, creates a GitHub release and uploads any `skip-export` artifacts

### Example: Framework with Custom Emulator Settings

```yaml
jobs:
  call-workflow:
    uses: skiptools/actions/.github/workflows/skip-framework.yml@v1
    with:
      emulator-api-level: '34'
      emulator-profile: 'medium_phone'
      run-ios-tests: false
```

---

## skip-app Workflow

A reusable workflow for building, testing, and deploying Skip application projects. It runs tests, builds debug and release exports, optionally signs the app for both platforms, and can submit to the Apple App Store and Google Play Store on semver tag pushes.

Use this workflow for Skip app projects (e.g., `skipapp-*` repositories).

### Usage

Create `.github/workflows/ci.yml` in your app repository:

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

### Inputs

| Input | Description | Type | Default |
|-------|-------------|------|---------|
| `brew-install` | Additional Homebrew packages to install before building | `string` | (none) |
| `run-local-tests` | Run local tests with `skip test` (skipped on tag pushes) | `boolean` | `true` |

### Secrets

These optional secrets enable app signing and store submission:

| Secret | Description |
|--------|-------------|
| `KEYSTORE_PROPERTIES` | Base64-encoded Android keystore properties file for APK/AAB signing |
| `KEYSTORE_JKS` | Base64-encoded Android keystore (`.jks`) file |
| `GOOGLE_PLAY_APIKEY` | Base64-encoded Google Play API key JSON for Play Store submission |
| `APPLE_APPSTORE_APIKEY` | Base64-encoded App Store Connect API key JSON for App Store submission |
| `APPLE_CERTIFICATES_P12` | Base64-encoded Apple signing certificate (`.p12`) |
| `APPLE_CERTIFICATES_P12_PASSWORD` | Password for the Apple signing certificate |
| `APPLE_MOBILEPROVISION` | Apple mobile provisioning profile |

### What It Does

1. **Environment setup**: Detects Skip Fuse mode from `skip.yml` files, sets the marketing version from the latest semver tag, and sets the build number from the git commit count
1. **Tests**: Runs `skip test` if a `Tests` directory exists (skipped on tag pushes)
1. **Export**: Runs `skip export` twice: once for debug and once for release, producing both iOS and Android build artifacts
1. **Android signing**: If `KEYSTORE_PROPERTIES` is provided, configures Android app signing with the keystore
1. **iOS signing**: If `APPLE_CERTIFICATES_P12` is provided, imports code signing certificates
1. **Fastlane**: Runs `fastlane assemble` in both `Android/` and `Darwin/` directories if Fastlane is configured (on non-tag pushes)
1. **App Store submission**: On semver tag pushes, submits to the Apple App Store and/or Google Play Store if the corresponding API key secrets are provided
1. **Release**: On semver tag pushes, creates a GitHub release with the exported artifacts
1. **Artifact upload**: Always uploads the `skip-export/` directory as a build artifact

### Example: App with Store Deployment

```yaml
jobs:
  call-workflow:
    uses: skiptools/actions/.github/workflows/skip-app.yml@v1
    secrets:
      KEYSTORE_PROPERTIES: ${{ secrets.KEYSTORE_PROPERTIES }}
      KEYSTORE_JKS: ${{ secrets.KEYSTORE_JKS }}
      GOOGLE_PLAY_APIKEY: ${{ secrets.GOOGLE_PLAY_APIKEY }}
      APPLE_APPSTORE_APIKEY: ${{ secrets.APPLE_APPSTORE_APIKEY }}
      APPLE_CERTIFICATES_P12: ${{ secrets.APPLE_CERTIFICATES_P12 }}
      APPLE_CERTIFICATES_P12_PASSWORD: ${{ secrets.APPLE_CERTIFICATES_P12_PASSWORD }}
```

---

## Releasing

To release a new version of these actions and update the symbolic `v1` tag:

```bash
git tag v1.0.0 && git push --tags && git tag -fa v1 -m "Update v1 tag" && git push origin v1 --force && gh release create --generate-notes --latest
```

