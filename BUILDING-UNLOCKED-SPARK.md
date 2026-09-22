# Building this fork

Fork of `google/talkback`, kept for accessibility research: this build is the screen reader
under which Unlocked Spark records what an application announces. It is not meant as a screen
reader for daily use, and it is not the reader people have on their phones, which matters when
reading any result produced with it.

## Changes against upstream

| Change | Why |
|---|---|
| `gradle/wrapper` and `gradlew` added | upstream ships no wrapper and expects `gradle` on the path; the wrapper pins 8.13, which is what the Android plugin 8.11.1 requires |
| `shared.gradle`: Kotlin compile tasks pinned to JVM target 17 | Java targets 17 while Kotlin follows the JDK that runs the build, which is 21 in the runtime bundled with Android Studio, and Gradle refuses a module whose compilers disagree |
| `braille/brltty` and `braille/translate`: `ndkVersion` 27.3.13750724 | NDK 21 has no arm64 host, so `ndk-build` fails on Apple Silicon with "Unknown host CPU architecture: arm64". The braille modules are not used by our work, but the service code refers to them, so they are built rather than removed |
| `skip_tutorial_in_launching` set to `true` | the first run tutorial takes over the screen and hides the application under test |

## Build

```
export ANDROID_SDK_ROOT="$HOME/Library/Android/sdk"
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
./gradlew assemblePhoneDebug
```

The apk lands in `build/outputs/apk/phone/debug/talkback-phone-debug.apk`. Its application id
is `com.android.talkback`, so it installs next to the system reader
(`com.google.android.marvin.talkback`) and either can be enabled.

Measured on 22 September 2026: built from the drop of 18 March 2026, release `TfPu_release_16_2`,
against SDK 36 with NDK 27.3.13750724 and the JDK 21 runtime from Android Studio. The build
takes about four minutes cold. Enabled on an emulator with our speech engine as the default,
it announced the fixture's unlabelled icon as "Button, Double-tap to activate, Labels available,
use Tap with 3 fingers to view", while the reader preinstalled on the emulator image says
"Unlabelled. Button" for the same element. Which reader produced a transcript therefore belongs
in the metadata of any run that quotes it.
