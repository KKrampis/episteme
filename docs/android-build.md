# Android Build

## GitHub Actions (CI)

The Android build runs automatically via GitHub Actions whenever code is pushed to `main` or any `claude/**` branch, and on pull requests targeting `main`.

Workflow file: `.github/workflows/android-build.yml`

### What it produces

A release APK built from the `standard` product flavor.

### Automatic GitHub Release (on push to `main`)

Every push to `main` triggers a build **and** publishes the APK to **GitHub Releases** automatically — no manual steps needed. The release is tagged `build-YYYYMMDD-<sha8>` (e.g. `build-20260714-a1b2c3d4`).

Find the latest APK at: **Repository → Releases** (right sidebar on the main page)

Releases are permanent and publicly downloadable — no GitHub account required, no expiry.

### Builds on `claude/**` branches and pull requests

These still compile and upload the APK as a workflow **artifact** (temporary, 30 days, requires GitHub login to download), but do **not** publish a GitHub Release. This keeps the Releases page clean.

### How to download the artifact (for non-main branches)

1. Go to the repository on GitHub → **Actions** tab
2. Click the build run
3. Scroll to **Artifacts** at the bottom
4. Download the zip and unzip — the `.apk` is inside

### Installing on an Android device

1. Enable **Install unknown apps** on your device:
   - Android 8+: Settings → Apps → Special app access → Install unknown apps → choose your file manager or browser → Allow
2. Transfer the APK to your phone (email, Google Drive, USB cable, etc.)
3. Tap the APK file to install

> **Note:** A release APK must be signed to install. A debug APK uses a generic debug key. If you have the Play Store version installed, Android may refuse to install over it due to a signing mismatch — uninstall the Play Store version first.

---

## Manual trigger

You can also trigger a build manually with a custom version tag:

1. Go to **Actions** → **Android build** → **Run workflow**
2. Select:
   - **Build type:** `debug` or `release`
   - **Flavor:** `standard` or `oss`
   - **Release version tag:** e.g. `1.2.0` (leave blank to use `build-YYYYMMDD-<sha>`)
3. Click **Run workflow**

A manual trigger always publishes to GitHub Releases.

---

## Release signing

Release builds are signed with your keystore if the following repository secrets are configured:

| Secret | Description |
|--------|-------------|
| `ANDROID_KEYSTORE_BASE64` | Base64-encoded `.keystore` file (`base64 your.keystore`) |
| `ANDROID_KEYSTORE_PASSWORD` | Keystore password |
| `ANDROID_KEY_ALIAS` | Key alias |
| `ANDROID_KEY_PASSWORD` | Key password |

To add secrets: **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

If the secrets are not set, a release build will still compile but will be unsigned (not installable on a device without manual signing).

---

## Build environment notes

GitHub Actions runners use **Temurin JDK 21**. The project's `gradle/gradle-daemon-jvm.properties` specifies a JetBrains JDK, so the workflow overrides it with:

```
toolchainVersion=21
```

This is done automatically by the workflow and does not affect local development.

### First build vs. cached builds

| | Cold (no cache) | Warm (cache hit) |
|-|----------------|-----------------|
| Build time | ~8–10 min | ~3–5 min |

Gradle dependency caches are stored between runs using `actions/cache`, keyed on the Gradle build files and `libs.versions.toml`.
