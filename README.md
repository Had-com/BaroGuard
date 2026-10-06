# BaroGuard

Barometric pressure monitor and alert app for Android. It reads the phone's built-in pressure sensor,
draws the history on an interactive graph and a home-screen widget, and warns you about rapid pressure
drops or rises, which can signal approaching weather fronts or trigger migraines and joint pain.

Everything is measured and stored on the phone. The only network use is the optional update check.

## Download

1. Open the [**Releases**](../../releases) page and pick the newest release.
2. Under **Assets**, download **`BaroGuard.apk`** and open it.
3. Allow installs from your browser if Android asks. If Play Protect warns about an unrecognised app, tap
   **More details → Install anyway**.

`BaroGuard.aab` in each release is the Play Console bundle and cannot be installed on a phone.

Requires Android 8.0 (API 26) or newer and a phone with a barometer. Without a pressure sensor the app shows
a "Barometer not available" screen.

## Features

**Monitoring**
- Background sampling every 5, 15, 30 or 60 minutes (WorkManager, low battery use).
- Units: hPa, mbar, inHg, mmHg.
- Readings are stored in a local Room database and removed after 30 days.

**Graph**
- Time ranges: 1h, 3h, 6h, 12h, 24h, 48h, 7d.
- Pinch to zoom, drag to pan, tap to inspect an exact time and value.
- The line is split into segments at every rise/fall turning point. Rising is red, falling is blue.
- Each segment shows its rate in the centre: 🔥 for a positive rate, ❄️ for a negative one.
- **Add mark** button records a timestamped mark (with an optional note, such as "headache") and the pressure
  at that moment. Marks show as dashed lines on the graph and can be deleted.

**Widget** (resizable, default 4x3)
- Current pressure, unit, trend, last-updated time and a refresh button.
- Graph with axes, values, units, segment rates, marks and the same colours as the app.
- Strong rises and drops (those that would trigger your alert) are highlighted.
- The latest reading sits at 3/4 of the x axis, with empty room on the right.
- 1h / 3h / 6h / 12h / 24h buttons on the widget itself.
- Background colour (presets or custom RGB) and opacity in Settings.

**Alerts**
- Rapid-drop and rapid-rise notifications when pressure changes by more than a threshold within a time window
  (default 2 hPa in 3 hours).
- Adjustable thresholds and window, sound, vibration, priority and quiet hours. System Do Not Disturb is respected.

**Updates**
- *Settings → App updates → Check for updates* shows what changed in every version you have not installed yet,
  and an **Install update** button downloads and installs it.

## How this repository builds

The repository holds `BaroGuard.zip` (the full Android Studio project) and a GitHub Actions workflow in
`.github/workflows/build.yml`. On every push the workflow:

1. unzips the project,
2. builds a signed release **APK** (direct install, with the in-app updater) and a signed **AAB** (Play build,
   without the updater and without network or install permissions),
3. publishes both as a new GitHub Release, tagged `v1.0.<run number>`. The run number is also the app's
   `versionCode`, so each build is newer than the last.

The release description comes from `RELEASE_NOTES.md` inside the project. The app shows that text in the update
check, so rewrite it for every version.

### Signing

Android only installs an update signed with the same key as the installed app.

- By default the build uses the fallback key stored in the project (`app/baroguard.keystore`). Because this
  repository is public, **that key is public**. It is fine for a personal app, but do not use it for the Play Store.
- To sign with a private key, add these repository secrets (*Settings → Secrets and variables → Actions*):

  | Secret | Value |
  |---|---|
  | `KEYSTORE_BASE64` | your keystore file, base64-encoded (`base64 -w0 release.jks`) |
  | `KEYSTORE_PASSWORD` | keystore password |
  | `KEY_ALIAS` | key alias |
  | `KEY_PASSWORD` | key password |

  Changing the signing key means existing installs must be uninstalled once before the new build installs.

### Building locally

Unzip `BaroGuard.zip`, open the folder in Android Studio and run it on a device with a barometer.
Emulators have no pressure sensor. From the command line:

```
gradle assembleDirectRelease   # APK
gradle bundlePlayRelease       # AAB for Google Play
```

## Project layout

```
app/src/main/java/com/baroguard/
  MainActivity.kt   Compose UI: graph, marks, settings, update section
  Monitoring.kt     sensor reading, scheduling, background worker, alert engine
  BaroWidget.kt     home-screen widget and its chart renderer
  Db.kt             Room database: readings and marks
  Prefs.kt          settings storage
  Units.kt          unit conversion, trend and segment calculation
  Updater.kt        GitHub Releases update check and installer
RELEASE_NOTES.md    summary of changes for the current version
```

## Tech

Kotlin, Jetpack Compose, Room, WorkManager, AppWidget. Min SDK 26, target SDK 36.

## Permissions

| Permission | Why |
|---|---|
| `POST_NOTIFICATIONS` | pressure alerts (Android 13+) |
| `RECEIVE_BOOT_COMPLETED` | restart sampling after a reboot |
| `INTERNET`, `REQUEST_INSTALL_PACKAGES` | update check and install (direct APK only, removed from the Play build) |
