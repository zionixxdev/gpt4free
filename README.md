# gpt4free — Android app (Chaquopy build)

Native Android app that embeds the g4f Python server via
[Chaquopy](https://chaquo.com/chaquopy). The Flask GUI (`g4f.gui`) runs
in-process and a WebView loads `http://127.0.0.1:1337/`.

This repo contains the **Android project only**. The `g4f` Python package is
not committed here — the CI workflow fetches it from upstream and places it at
`./g4f` before building (see `.github/workflows/build-apk.yml`).

## Build the APK (from a phone — no PC needed)

1. Open the **Actions** tab and run **Build Android APK**
   (or just push a commit to `main`).
2. When the run finishes, open it and download the **`g4f-debug-apk`**
   artifact (a `.zip`).
3. Unzip it on the phone, tap the `.apk`, and allow "Install unknown apps".

## Layout

```
.github/workflows/build-apk.yml   CI that builds the APK
android/                          Gradle project root
  app/build.gradle                app config + Chaquopy pip packages
  app/src/main/java/.../MainActivity.java
  app/src/main/python/bootstrap.py
g4f/                              fetched by CI at build time (git-ignored)
```

## Build locally (if you have a desktop)

Toolchain: JDK 17, Android SDK 34, Gradle 8.7 (bundled via the wrapper).

```bash
# place the g4f package next to the android project
git clone --depth 1 https://github.com/xtekky/gpt4free.git _g4f_src
cp -r _g4f_src/g4f ./g4f

cd android
./gradlew assembleDebug
# output: app/build/outputs/apk/debug/app-debug.apk
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## Notes on the ABI configuration

- Bundled ABIs: `arm64-v8a`, `armeabi-v7a`, `x86_64`.
- The 32-bit `armeabi-v7a` ABI requires **Python 3.11** — Chaquopy supports
  32-bit only up to Python 3.11 (3.12+ is 64-bit only), so the Chaquopy
  runtime is pinned to `3.11` in `app/build.gradle`. Do not raise it to 3.12
  while `armeabi-v7a` is in `abiFilters`.
- Native pip packages (`aiohttp`, `brotli`, `pycryptodome`, `pillow`,
  `cryptography`) must have `armeabi-v7a` wheels in Chaquopy's repository; the
  build will name any package that doesn't.
