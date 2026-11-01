# NOVA Android — Build APK with GitHub Actions

This is an Android Studio wrapper for NOVA. The current web UI is bundled unchanged apart from video integration text/logic. Android copies selected videos into app-private storage and opens them with Android's system player chooser. VLC/MX Player appear only when installed and registered for the selected video's MIME type.

## Upload and build
1. Create a **Public** GitHub repository.
2. Extract this ZIP and upload the contents of `nova_android_github` to the repository root, including the hidden `.github` folder. `app/`, `.github/`, `build.gradle`, `settings.gradle`, and `gradle.properties` must be at the root.
3. Commit the files. The workflow runs automatically on the first push.
4. Open **Actions** and inspect `Build NOVA Android APK`. On success, download **Artifacts → NOVA-Android-APK**.
5. To create a permanent Releases download, publish a release tagged `v1.0.0`. The workflow attaches the APK. Use a new tag such as `v1.0.1` for later versions.

## Stable signing for app updates (recommended)
Android only installs an update over an existing app when the signing certificate matches. Before installing a long-term build, create a private keystore once:

```bash
keytool -genkeypair -v -keystore nova-release.jks -alias nova -keyalg RSA -keysize 2048 -validity 10000
```

In PowerShell, copy the keystore as Base64:

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("nova-release.jks")) | Set-Clipboard
```

In GitHub, open **Settings → Secrets and variables → Actions** and add these repository secrets: `NOVA_KEYSTORE_BASE64` (the Base64 value), `NOVA_STORE_PASSWORD`, `NOVA_KEY_ALIAS` (usually `nova`), and `NOVA_KEY_PASSWORD`. Never commit or upload the keystore itself. Keep a private backup; losing it prevents future signed updates.

When all four secrets are configured, the workflow creates a release-signed APK with the same key on every build and increments `versionCode` automatically. Without the secrets, it creates a debug-signed test APK; a later CI run may have a different debug certificate and cannot update the earlier install. Uninstalling to switch signatures removes NOVA's private video copies.

## Chrome download
Open **Releases → latest release → APK asset → Download**. Open the downloaded APK and follow Android's installation prompts. You may need to allow Chrome to install apps from this source.

## Implementation notes and limitations
- Bundled assets are served by AndroidX WebViewAssetLoader at `https://appassets.androidplatform.net/`, enabling a secure origin for IndexedDB and local storage.
- Selected videos are copied as streams to app-private storage; the app does not Base64-encode the entire file or load it all into RAM. The phone needs enough free internal storage.
- The external chooser can open VLC/MX Player only when compatible players are installed. Playback of every codec, resolution, and hardware combination cannot be guaranteed.
- Uninstalling NOVA or clearing its app data removes private video copies. Deleting the Gallery original does not remove NOVA's independent copy while app data remains intact.
- This environment did not run an Android SDK/Gradle build or test on a physical phone. A successful GitHub Actions build and real-device VLC/MX Player test are still required.
