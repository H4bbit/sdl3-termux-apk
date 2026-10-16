# SDL3 Termux Android

Native SDL3 Android build pipeline running entirely inside Termux without Android Studio.

This project demonstrates how to:
- Build native SDL3 applications for Android
- Compile C code with the Android NDK
- Package APKs manually
- Compile Java sources manually
- Generate DEX files manually
- Link Android resources manually
- Sign APKs manually
- Work with SDL3 Android integration without Gradle or Android Studio

Everything is built directly inside Termux (aarch64) using shell
scripts and command line tools. This project targets the Termux app
from GitHub releases only (the Play Store build is deprecated and
unsupported) — paths, toolchain and packaging choices below assume
that environment.

---

## Overview

This repository is intentionally minimal and low-level. Instead of relying on:
- Android Studio
- Gradle
- Java/Kotlin project generators
- SDL template generators

This project exposes the actual Android native build pipeline step-by-step. 

The goal is educational and practical:

- Understand how Android native packaging works
- Understand SDL3 Android integration
- Understand manual APK assembly
- Allow Android native development directly on-device inside Termux

This project intentionally avoids:

- Gradle
- Android Studio
- ndk-build
- Java/Kotlin Android application templates
- Android SDK binaries (aapt2, d8, zipalign, apksigner from the SDK
  do not execute correctly inside Termux — the Termux-packaged
  equivalents are used instead)

The entire APK is assembled manually through shell scripts and command-line tooling.


---

## Features

- SDL3 + Android
- Pure C application
- SDL3 callback entrypoints (`SDL_AppInit`, `SDL_AppEvent`, etc.)
- Manual APK packaging
- Manual DEX generation
- Native `.so` injection
- Touch input support
- Multi-touch particle demo
- No Gradle
- No Android Studio
- Works fully inside Termux from GitHub releases (aarch64-only, Play Store build unsupported)

---

## Current Demo

The current example is a simple SDL3 particle system:

- Fullscreen renderer
- Multi-touch support
- Touch trails
- Randomized particles
- Gravity simulation
- Lifetime fading

Implemented in:

```text
src/main.c
```


---

## Project Structure

```text
 .
 ├── build.sh
 ├── clean.sh
 ├── gen-debug-key.sh
 ├── CMakeLists.txt
 ├── LICENSE.txt
 ├── README.md
 ├── platform/
 │   └── android/
 │       └── app/
 │           ├── AndroidManifest.xml
 │           ├── java/
 │           └── res/
 ├── src/
 │   └── main.c
 ├── vendor/
 │   └── SDL/   (SDL3 checkout, not included — see below)
 ├── build-android/   (native CMake build, ignored)
 ├── build-java/      (javac classes + dex, ignored)
 ├── build-apk/       (reszip, base, inject, final APK, ignored)
 └── build/
     └── build-apk/
         └── debug.keystore

```


---

## Important Design Decisions

### Why no Android Studio?
This project exists specifically to prove that Android native applications can be built entirely inside Termux. Using Android Studio would defeat the purpose.

### Why manual APK packaging?
Normally Android builds are abstracted behind Gradle, Android Studio, and build plugins. This repository intentionally exposes the actual build pipeline:

* C → shared library (.so)
* Java → .class
* .class → DEX
* Resources → compiled resources
* Everything → APK
* APK → aligned
* APK → signed

This makes the Android native stack significantly easier to understand.

### Why Termux packages instead of Android SDK binaries?
Several Android SDK binaries do not execute correctly inside Termux because they were built expecting a conventional Linux environment. For this reason, this project uses:

- aapt2
- d8
- zipalign
- apksigner

from the Termux repositories instead of the Android SDK versions. Java tooling is also aligned around the Termux OpenJDK environment.

The only piece required from the Android SDK itself is the platform
stub (`platforms/android-34/android.jar`), which contains no binaries
and is consumed as a compile/link reference. It can be installed with
the SDK command-line tools (`sdkmanager "platforms;android-34"`).

### Why Java 17?
The Termux Android tooling ecosystem works reliably with OpenJDK 17. The build pipeline was designed around openjdk-17 inside Termux.

## Dependencies

| Dependency | Minimum | Notes |
|---|---|---|
| Termux (aarch64) | — | `$PREFIX` environment, `pkg` packages below |
| SDL3 | `main` at time of build | cloned into `vendor/SDL`, provides `android-project/` |
| Android SDK platform | `android-34` | only `platforms/android-34/android.jar` is used |
| Termux-adapted NDK | r29 (aarch64) | provides toolchain + `libc++_shared.so` |
| openjdk-17 | 17 | `javac`, `keytool` (via `pkg`) |
| cmake, ninja, aapt2, d8, apksigner, zipalign, zip | — | all via `pkg` |

## Termux Setup
Install required packages:
```bash
pkg update
pkg install \
  git \
  cmake \
  ninja \
  openjdk-17 \
  aapt2 \
  d8 \
  apksigner \
  zipalign \
  zip

```

## Android SDK Setup
The build expects two environment variables pointing wherever you
installed each piece (no fixed location required):

- `ANDROID_HOME` — an Android SDK whose `platforms/android-34/`
  holds `android.jar` (only this stub is used from the SDK).
  Install it with the SDK command-line tools, e.g.:

```bash
sdkmanager "platforms;android-34"
```

- `ANDROID_NDK_HOME` — the extracted Termux-adapted NDK root
  (the directory containing
  `build/cmake/android.toolchain.cmake`).

```bash
export ANDROID_HOME=<your-sdk-dir>
export ANDROID_NDK_HOME=<your-ndk-dir>
```
Only the platform stub is used from the SDK — packaging tools
(aapt2, d8, zipalign, apksigner) come from the Termux packages above.
## Android NDK

This project uses `termux-ndk`.

The official Android NDK does not integrate cleanly with Termux in many cases. The Termux-adapted NDK avoids several compatibility issues.

Download the Termux-adapted NDK tarball (do NOT clone the repo — it does not contain a usable NDK tree) and extract it to a directory of your choice:

```bash
mkdir -p <ndk-parent-dir>
curl -sL https://github.com/lzhiyong/termux-ndk/releases/download/android-ndk/android-ndk-r29-aarch64.tar.xz \
  | tar -xJ -C <ndk-parent-dir>
```

Then set `ANDROID_NDK_HOME` to the extracted NDK root — the
directory containing `build/cmake/android.toolchain.cmake` (check
with `ls`, as the tarball creates a versioned subdirectory).
## SDL3 Dependency

SDL is intentionally not included in this repository.

Clone SDL3 into `vendor/SDL`:

```bash
git clone https://github.com/libsdl-org/SDL.git vendor/SDL
```

This repository expects:

```text
vendor/SDL/android-project/
```

to exist.

Official SDL repository:
https://github.com/libsdl-org/SDL

---

## Build Instructions

### Generate Debug Keystore
Before building:
```bash
chmod +x gen-debug-key.sh
./gen-debug-key.sh

```
This generates: `build/build-apk/debug.keystore`

### Build APK
```bash
chmod +x build.sh
./build.sh

```
Final APK output:

```text
build-apk/final/app-signed.apk
```

Intermediate outputs: `build-android/` (native `.so`),
`build-java/` (`classes`, `dex`), `build-apk/` (`reszip`,
`base`, `inject`, `final`).

### Clean Build Files
```bash 
chmod +x clean.sh
./clean.sh
```
### Native Build Pipeline

The build script performs the following steps:

1. Configure CMake for Android
2. Compile the app `.so` against static SDL3 (`SDL_STATIC ON` — no shared SDL library is built)
3. Compile Java sources (SDL `SDLActivity` + app `MainActivity`)
4. Generate DEX bytecode
5. Compile Android resources
6. Link the base APK
7. Inject native `.so` libraries (`libsdl-android-app.so` +
   `libc++_shared.so` from the NDK, host `linux-aarch64` —
   Termux-only scope) and `classes.dex`
8. Align APK
9. Sign APK


---

## License

This project is licensed under the MIT License.

See `LICENSE.txt` for details.
