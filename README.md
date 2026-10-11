# Galaxy Bio: Galaxy Watch 8 sensors on any Android phone

Two apps in one project:

| Module | Runs on | Does |
|---|---|---|
| `app` | Galaxy Watch 8 | Reads heart rate (Google Health Services), SpO2 and ECG (Samsung Health Sensor SDK). Works alone, and also sends every reading to the phone. |
| `mobile` | Your Android phone (any brand) | Shows live heart rate, last SpO2 and ECG, a recent-readings list, and saves everything to a CSV you can share. |

Both apps use the same `applicationId` (`com.example.galaxybio`) and **must be signed with the same key**. That is what lets Wear OS's Data Layer pass messages between them.

## Get the APKs without installing anything (GitHub Actions)

1. Create a GitHub repo and upload everything in this folder, including the hidden `.github` folder.
2. Open the repo's **Actions** tab, run **Build APKs** (it also runs on every push).
3. Download two artifacts: `galaxybio-phone-apk` and `galaxybio-watch-apk`.

Both are built in one run with the same debug key, so they can talk to each other. If you build them separately or on different machines, the keys differ and the phone will not receive anything.

The CI build is the **stub** watch build: heart rate only. For SpO2 and ECG see "Samsung SDK" below.

## Install

**Phone:** open the phone APK on the phone and allow installing from that source.

**Watch** (no way around adb for a watch): Settings > About watch > Software, tap *Software version* 7 times. Then Settings > Developer options > turn on **ADB debugging** and **Debug over Wi-Fi**. On a computer with adb: `adb pair <ip>:<pairing-port>`, then `adb connect <ip>:<port>`, then `adb install app-stub-debug.apk`.

The watch must be paired to the phone through the **Galaxy Wearable** app (works on non-Samsung Android phones). Keep Bluetooth on.

## Use it

Open Galaxy Bio on the watch, tap *Heart rate*. Open Galaxy Bio on the phone. Numbers appear within a few seconds. The phone app does not need to be open for readings to be saved; it is woken when the watch sends one.

## Samsung SDK (SpO2 and ECG)

1. Download the Health Sensor SDK `.aar` (v1.4.x) from https://developer.samsung.com/health/sensor and put it in `app/libs/`.
2. Build the `samsungDebug` variant of the watch app: `gradle :app:assembleSamsungDebug` (or in Android Studio). To keep the same signing key as the phone app, build both from the same machine: `gradle :mobile:assembleDebug :app:assembleSamsungDebug`.
3. On the watch, enable Samsung developer mode: Settings > Apps > **Health Sensor Service** (or *Health Platform*) > tap the title about 10 times > **Developer mode** > accept.

## Things to know

- Personal use only. Samsung requires partner approval to distribute an app that uses its SDK.
- Not a medical device. The ECG summary is a rough estimate. The SDK does not offer blood pressure.
- Samsung SDK code was written from Samsung's docs and has not been compiled against the real `.aar`. If a name differs in your version, the compiler will show the exact line.
- None of this has been built or run on a device yet. Expect to fix a small error or two on first build.
- Data Layer needs Google Play services on the phone. Most non-Samsung Android phones outside China have it.
- Readings are only sent while the watch app is open and the phone is connected. Nothing is queued for later.
