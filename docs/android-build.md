# Android Build

## GitHub Actions (CI)

The Android build runs automatically via GitHub Actions whenever code is pushed to `main` or any `claude/**` branch, and on pull requests targeting `main`.

Workflow file: `.github/workflows/android-build.yml`

### What it produces

A debug APK built from the `standard` product flavor. The APK is uploaded as a workflow artifact and kept for 30 days.

### How to download the APK (artifact)

1. Go to the repository on GitHub → **Actions** tab
2. Click the latest **Android build** run
3. Scroll to the bottom → **Artifacts**
4. Download the zip (e.g. `episteme-debug-standard-<sha>`)
5. Unzip — the `.apk` file is inside

> Artifacts expire after 30 days and require a GitHub login to download.

### Installing on an Android device

1. Enable **Install unknown apps** on your device:
   - Android 8+: Settings → Apps → Special app access → Install unknown apps → choose your file manager or browser → Allow
2. Transfer the APK to your phone (email, Google Drive, USB cable, etc.)
3. Tap the APK file to install

> **Note:** A debug APK is signed with a generic debug key. If you have the Play Store version installed, Android may refuse to install over it due to a signing mismatch — uninstall the Play Store version first.

---

## Manual / Release Builds

The workflow supports a manual trigger (`workflow_dispatch`) for building release APKs and optionally publishing them to **GitHub Releases** — permanent, public download links that do not expire and do not require a GitHub login.

### How to manually trigger a release build and publish it

1. Go to the repository on GitHub → **Actions** tab
2. In the left sidebar click **Android build**
3. Click the **Run workflow** button (top right of the runs list)
4. Fill in the inputs:

   | Input | What to set |
   |-------|-------------|
   | **Build type** | `release` (or `debug` for testing) |
   | **Product flavor** | `standard` or `oss` |
   | **Publish to GitHub Releases** | `true` to create a public release |
   | **Release version tag** | A version string like `1.0.0` — this becomes the Git tag `v1.0.0` and is part of the APK filename |

5. Click **Run workflow**

The build takes ~3–10 minutes. When it finishes:
- The APK is always uploaded as a workflow artifact (temporary, 30 days).
- If you set **Publish to GitHub Releases** = `true` and supplied a version, the APK is also published to **Releases** → permanently accessible at a public URL like `https://github.com/<owner>/episteme/releases/tag/v1.0.0`.

### Finding the released APK

Go to the repository → **Releases** (right sidebar on the main page, or `https://github.com/<owner>/episteme/releases`). Click the release to expand the assets and download the APK directly — no GitHub account required.

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

This is done automatically by the workflow and does not affect local development (the file is restored after the workflow step).

### First build vs. cached builds

| | Cold (no cache) | Warm (cache hit) |
|-|----------------|-----------------|
| Build time | ~8–10 min | ~3–5 min |

Gradle dependency caches are stored between runs using `actions/cache`, keyed on the Gradle build files and `libs.versions.toml`.
