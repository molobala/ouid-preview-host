# OUID Preview Host Android Releases

This repository stores Android preview host APKs consumed by OUID IDEX.

GitHub repository:

```text
https://github.com/molobala/ouid-preview-host.git
```

The preview host lets a mini-app developer install a generic OUIDRuntime host on a real Android device, push a local preview bundle into the app sandbox, then launch the mini app without going through the backend validation flow.

## Build From The Preview Host App

From the `ouid-preview-host` Flutter project:

```sh
bash scripts/build_android_releases.sh
```

This writes:

```text
releases/
  manifest.json
  ouid-preview-host-release-arm64-v8a.apk
  ouid-preview-host-release-armeabi-v7a.apk
  ouid-preview-host-release-x86_64.apk
```

Use the default `MODE=release` for hosted artifacts. OUID IDEX downloads these APKs into its user-data cache and serves direct USB preview bundles through `adb reverse`, so APKs do not need to be embedded in IDE resources and the Preview Host does not need to be debuggable.

Optional build variables:

```sh
BUILD_NAME=1.0.1 BUILD_NUMBER=2 bash scripts/build_android_releases.sh
```

For local Preview Host development, build debug APKs outside this `releases` git repository and point the IDE to the matching ABI APK:

```sh
MODE=debug RELEASES_DIR=/tmp/ouid-preview-host-debug bash scripts/build_android_releases.sh
OUID_PREVIEW_HOST_DEBUG_APK=/tmp/ouid-preview-host-debug/ouid-preview-host-debug-arm64-v8a.apk npm run dev
```

Debug APKs are intentionally not tracked in this repository because they exceed GitHub's normal git file size limit. If debug APKs need to be shared between internal developers, host them as external release assets and add them to `android.debugArtifacts` in `manifest.json`.

The build script defaults artifact URLs to:

```sh
https://raw.githubusercontent.com/molobala/ouid-preview-host/main
```

Override `BASE_URL` only when publishing to another artifact repository:

```sh
BASE_URL=https://raw.githubusercontent.com/<org>/<repo>/main bash scripts/build_android_releases.sh
```

## Manifest

`manifest.json` is the stable entry point for OUID IDEX.

The IDE downloads the manifest, detects the connected Android device ABI with `adb`, selects the matching artifact from `android.artifacts`, verifies `sha256`, installs the APK when needed, then serves the mini-app bundle through a temporary `adb reverse` preview URL.

Example:

```json
{
  "version": "1.0.0+1",
  "generatedAt": "2026-05-16T00:00:00Z",
  "android": {
    "packageName": "com.ouidesigner.previewhost",
    "mode": "release",
    "artifacts": [
      {
        "abi": "arm64-v8a",
        "targetPlatform": "android-arm64",
        "apkUrl": "./ouid-preview-host-release-arm64-v8a.apk",
        "fileName": "ouid-preview-host-release-arm64-v8a.apk",
        "sha256": "...",
        "size": 12345678,
        "version": "1.0.0+1"
      }
    ]
  }
}
```

## Publishing

This folder is intentionally ignored by the parent Flutter project. Initialize this directory as its own Git repository for GitHub hosting:

```sh
git init
git add README.md manifest.json *.apk
git commit -m "Publish Android preview host"
git remote add origin https://github.com/molobala/ouid-preview-host.git
git push -u origin main
```

OUID IDEX uses this raw GitHub manifest URL by default:

```text
https://raw.githubusercontent.com/molobala/ouid-preview-host/main/manifest.json
```

Use `OUID_PREVIEW_HOST_MANIFEST_URL` only to override the default source:

```sh
OUID_PREVIEW_HOST_MANIFEST_URL=https://raw.githubusercontent.com/<org>/<repo>/main/manifest.json
```
