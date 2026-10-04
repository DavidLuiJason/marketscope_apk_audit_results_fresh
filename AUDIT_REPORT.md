# MarketScope APK Build Audit

## Scope

This audit checks whether the supplied MarketScope project can be taken from its source ZIP through GitHub Actions to a real, installable Android APK.

The original application files were not modified, repaired, renamed, reorganized, or deleted for this audit.

## Confirmed findings

### 1. Missing native Android source files — BLOCKER

The GitHub build process invokes:

`node scripts/patch-android.mjs`

That script expects these files:

- `native/android/MainActivity.java`
- `native/android/CollectorService.java`
- `native/android/NativeBridgePlugin.java`

Those files are absent from the supplied project ZIP.

As a result, the Android patch stage is expected to fail before the APK can be produced.

### 2. Dependency lockfile mismatch — BUILD RELIABILITY ISSUE

The project contains `bun.lock`, while the GitHub workflow uses npm installation rather than a frozen installation matching that Bun lockfile.

This means the exact dependency versions represented by the supplied lockfile are not necessarily what the GitHub runner will install.

For reproducible builds, the project should use one package manager consistently and have the workflow use the corresponding lockfile.

### 3. Native bridge implementation cannot be verified

The web application contains a native bridge interface that expects Android-side functionality including service control, notification permission handling, battery status/settings, and status retrieval.

Because the required native Java source files are absent, the corresponding native implementation and registration cannot be verified from the supplied ZIP.

## Architecture that is already present

The project already contains the major pieces of a legitimate APK pipeline, including:

- React / TypeScript / Vite
- Capacitor packages
- `capacitor.config.json`
- Android build workflow
- Android/Gradle build commands
- APK artifact upload configuration
- MarketScope application name
- Android application ID
- `dist` web output configuration

Therefore this is not a case of needing to convert an ordinary website from scratch. The project already has an intended Capacitor-to-Gradle APK pipeline.

## Important preservation requirement

This audit does NOT authorize restructuring or redesigning the application.

Any future repair should preserve the existing application source, UI, assets, business logic, configuration, and project structure unless a specific Android build requirement requires a narrowly scoped addition or correction.

Do not solve build failures by inserting placeholder native implementations that merely make the build pass while leaving application functionality broken.

## Result

The supplied ZIP is NOT currently verified as APK-build-ready.

The primary confirmed blocker is the missing native Android source expected by the project's own patch script. The dependency-locking inconsistency is a secondary reproducibility problem.

The included `ORIGINAL_SHA256SUMS.txt` records SHA-256 hashes of every original project file extracted from the supplied ZIP so the source contents can be independently checked for alteration.
