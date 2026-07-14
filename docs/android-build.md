# Android Build

## GitHub Actions (CI)

The Android build runs automatically via GitHub Actions whenever code is pushed to `main` or any `claude/**` branch, and on pull requests targeting `main`.

Workflow file: `.github/workflows/android-build.yml`

### What it produces

A debug APK built from the `standard` product flavor. The APK is uploaded as a workflow artifact and kept for 30 days.

### How to download the APK

1. Go to the repository on GitHub → **Actions** tab
2. Click the latest **Android build** run
3. Scroll to the bottom → **Artifacts**
4. Download the zip (e.g. `episteme-debug-standard-<sha>`)
5. Unzip — the `.apk` file is inside

### Installing on an Android device

1. Enable **Install unknown apps** on your device:
   - Android 8+: Settings → Apps → Special app access → Install unknown apps → choose your file manager or browser → Allow
2. Transfer the APK to your phone (email, Google Drive, USB cable, etc.)
3. Tap the APK file to install

> **Note:** A debug APK is signed with a generic debug key. If you have the Play Store version installed, Android may refuse to install over it due to a signing mismatch — uninstall the Play Store version first.

---

## Manual / Release Builds

The workflow also supports a manual trigger (`workflow_dispatch`) for building release APKs.

### Triggering a manual build

1. Go to **Actions** → **Android build** → **Run workflow**
2. Select:
   - **Build type:** `debug` or `release`
   - **Flavor:** `standard` or `oss`
3. Click **Run workflow**

### Release signing

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
