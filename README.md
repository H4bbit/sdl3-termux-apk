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

Everything is built directly inside Termux using shell scripts and command line tools.

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
- Works fully inside Termux

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
 └── vendor/
     └── SDL -> ../../SDL

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

This project depends on:

- SDL3
- Android SDK
- Android NDK
- Termux packages
- Ninja
- CMake

## Termux Setup
Install required packages:
```bash
pkg update
pkg install \
  git \
  cmake \
  ninja \
  clang \
  openjdk-17 \
  aapt \
  aapt2 \
  d8 \
  apksigner \
  zipalign

```

## Android SDK Setup
The build expects an Android SDK providing the API 34 platform stub
(`platforms/android-34/android.jar`), pointed to by `ANDROID_HOME`, and
the Termux-adapted NDK, pointed to by `ANDROID_NDK_HOME`, e.g.:
```bash
export ANDROID_HOME=$HOME/path/to/android-sdk
export ANDROID_NDK_HOME=$HOME/path/to/android-ndk

```
Only the platform stub is used from the SDK — packaging tools
(aapt2, d8, zipalign, apksigner) come from the Termux packages above.
## Android NDK

This project uses `termux-ndk`.

The official Android NDK does not integrate cleanly with Termux in many cases. The Termux-adapted NDK avoids several compatibility issues.

Download the Termux-adapted NDK tarball (do NOT clone the repo — it does not contain a usable NDK tree) and extract it somewhere accessible, for example:

```bash
mkdir -p ~/android-ndk
curl -sL https://github.com/lzhiyong/termux-ndk/releases/download/android-ndk/android-ndk-r29-aarch64.tar.xz \
  | tar -xJ -C ~/android-ndk
```
## SDL3 Dependency

SDL is intentionally not included in this repository.

Clone SDL3 manually into `vendor/SDL`:

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
This generates: build/build-apk/debug.keystore

### Build APK
```bash
chmod +x build.sh
./build.sh

```
Final APK output:

```text
build-apk/final/app-signed.apk
``` 

### Clean Build Files
```bash 
chmod +x clean.sh
./clean.sh
```
### Native Build Pipeline

The build script performs the following steps:

1. Configure CMake for Android
2. Compile native SDL3 shared libraries
3. Compile Java sources
4. Generate DEX bytecode
5. Compile Android resources
6. Link the base APK
7. Inject native `.so` libraries and `classes.dex`
8. Align APK
9. Sign APK


---

## License

This project is licensed under the MIT License.

See `LICENSE.txt` for details.
