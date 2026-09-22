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
| `talkback/adb/` package plus four hooks in the service | lets the reader be driven from the command line, see below |

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

## Driving the reader from the command line

A screen reader used as a measuring instrument has to be told where to go, and synthetic touch
events do not drive one: its gestures carry timing and multi finger thresholds that `adb shell
input` cannot reproduce. The `talkback/adb` package answers that with a broadcast receiver,
registered when the service connects.

```
adb shell am broadcast -a com.a11y.adb.first_in_screen
adb shell am broadcast -a com.a11y.adb.next
adb shell am broadcast -a com.a11y.adb.next -e mode headings
adb shell am broadcast -a com.a11y.adb.perform_click_action
adb shell am broadcast -a com.a11y.adb.volume_max
```

Actions come from `A11yAction`: next and previous with a granularity, first and last in screen,
click and long click, scrolling, window navigation, the menus, reading from the top. Developer
toggles and the accessibility volume are there as well.

The code comes from TalkBack for Developers (github.com/qbalsdon/talkback, Apache 2.0) and is
carried onto this drop with two things left out: the slider action and the block out overlay,
both of which lean on code that exists only in that fork. The hooks it needs are an import, two
methods on the service, a register and unregister call, and `SelectorController.moveAtGranularity`
made visible to the service.

**What this measures and what it does not.** The receiver asks the service to run the action a
gesture is bound to, so it answers what the reader reads and in what order. It says nothing
about whether an application's own views break exploration by touch; that needs multi touch
injection on real hardware.

Measured on the fixture's icon screen, 22 September 2026: the reader goes title, heading, the
two icons under it, the next heading, then its icons, leaving the separator and the image last,
and wraps to the top afterwards. A walk that sets the accessibility focus from the tree instead
visits the separator and the image before the icons, which is why the order has to come from the
reader and not from us.
