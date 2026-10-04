# Native Android & iOS for React Native / Expo Developers

> A practical from-scratch guide to understanding `/android` and `/ios`,
> native builds, Gradle, Xcode, CocoaPods, where native
> code/configuration lives, what happens in the background, and how to
> debug problems quickly.

------------------------------------------------------------------------

## 1. The mental model

An Expo/React Native application has two major layers:

``` text
React / TypeScript
       |
       v
React Native
       |
       +-------------------+
       |                   |
       v                   v
    Android              iOS
       |                   |
     Gradle              Xcode
       |                   |
 Android SDK          Apple SDK
       |                   |
     APK/AAB          .app/.ipa
```

Expo does not replace Gradle or Xcode. Expo/EAS automates and
orchestrates native tooling.

A production build is roughly:

``` text
JS/TS source
   ↓
Metro bundles JavaScript
   ↓
React Native native project
   ↓
Native dependencies
   ↓
Android: Gradle
iOS: CocoaPods + Xcode
   ↓
Compile + link + package + sign
   ↓
Android APK/AAB or iOS .app/IPA
```

This distinction is essential when debugging.

------------------------------------------------------------------------

# 2. What `/android` and `/ios` are

If your project contains:

``` text
my-app/
├── app/
├── src/
├── assets/
├── package.json
├── android/
└── ios/
```

then:

``` text
android/ = the native Android application
ios/     = the native iOS application
```

Your React code is not compiled directly into an APK or IPA.

It is packaged into a native application that contains:

-   React Native runtime
-   JavaScript bundle
-   native modules
-   platform resources
-   platform configuration
-   native SDKs

------------------------------------------------------------------------

# 3. Expo prebuild and native folders

A managed-style Expo project may initially have:

``` text
package.json
app.json
app/
assets/
```

and no `android/` or `ios/`.

Running:

``` bash
npx expo prebuild
```

generates native projects.

After that you can work directly with:

``` bash
cd android
./gradlew assembleRelease
```

and:

``` bash
cd ios
pod install
```

followed by Xcode/xcodebuild.

### Important

Expo config such as:

``` text
app.json
app.config.js
app.config.ts
```

and Expo config plugins can modify generated native files.

Therefore there can be two sources you must understand:

``` text
Expo configuration
       ↓
Generated native configuration
```

If native folders are regenerated, manual changes can be overwritten.
For permanent generated-project changes, use appropriate config/plugin
mechanisms.

------------------------------------------------------------------------

# 4. Android folder structure

A typical React Native Android project looks like:

``` text
android/
├── app/
│   ├── build.gradle
│   ├── proguard-rules.pro
│   └── src/
│       ├── debug/
│       │   └── AndroidManifest.xml
│       ├── main/
│       │   ├── AndroidManifest.xml
│       │   ├── java/
│       │   │   └── com/example/app/
│       │   │       ├── MainActivity.kt
│       │   │       └── MainApplication.kt
│       │   └── res/
│       │       ├── drawable/
│       │       ├── mipmap-hdpi/
│       │       ├── mipmap-mdpi/
│       │       ├── mipmap-xhdpi/
│       │       ├── mipmap-xxhdpi/
│       │       ├── mipmap-xxxhdpi/
│       │       ├── values/
│       │       │   ├── strings.xml
│       │       │   ├── colors.xml
│       │       │   └── themes.xml
│       │       └── xml/
│       └── release/
│           └── AndroidManifest.xml
├── build.gradle
├── gradle.properties
├── gradlew
├── gradlew.bat
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── settings.gradle
└── local.properties
```

The exact structure varies with React Native/Expo/Gradle versions.

------------------------------------------------------------------------

# 5. `android/gradlew`

`gradlew` is the Gradle Wrapper executable for macOS/Linux.

Run:

``` bash
./gradlew assembleDebug
```

instead of depending on a globally installed Gradle.

The wrapper uses the Gradle version declared by:

``` text
gradle/wrapper/gradle-wrapper.properties
```

Windows uses:

``` bash
gradlew.bat
```

------------------------------------------------------------------------

# 6. `android/gradle/wrapper/gradle-wrapper.properties`

This selects the Gradle distribution.

You will normally see something like:

``` properties
distributionUrl=...gradle-<version>-bin.zip
```

If you get:

``` text
Unsupported Gradle version
```

or an Android Gradle Plugin/Gradle compatibility error, inspect:

``` text
gradle-wrapper.properties
android/build.gradle
settings.gradle
```

------------------------------------------------------------------------

# 7. `android/settings.gradle`

This is a central Gradle project configuration file.

It describes:

-   included modules
-   plugin management
-   repositories
-   React Native Gradle integration
-   native module/autolinking configuration in modern React Native
    projects

A key concept is:

``` gradle
include(":app")
```

which means the `app` module belongs to the Android project.

When you see:

``` text
plugin not found
autolinking failed
module not found
Gradle project configuration error
```

inspect this file.

------------------------------------------------------------------------

# 8. `android/build.gradle`

This is the root Android project configuration.

It is different from:

``` text
android/app/build.gradle
```

The root file generally controls project-wide build plugins,
repositories and shared configuration.

Think:

``` text
android/build.gradle
        |
        +---- project-level configuration
        |
        +---- app/build.gradle
        +---- other modules
```

------------------------------------------------------------------------

# 9. `android/app/build.gradle`

This is one of the most important files.

It configures the actual application module.

Important settings can include:

``` text
namespace
applicationId
compileSdk
minSdk
targetSdk
versionCode
versionName
buildTypes
productFlavors
signing
dependencies
React Native Gradle plugin
R8/ProGuard
packaging
```

Example:

``` gradle
android {
    namespace "com.example.app"

    defaultConfig {
        applicationId "com.example.app"
        minSdk ...
        targetSdk ...
        versionCode 1
        versionName "1.0"
    }
}
```

If you need to understand how Android produces your application, learn
this file very well.

------------------------------------------------------------------------

# 10. `applicationId`

Example:

``` gradle
applicationId "com.rslcards.dealer"
```

This identifies the Android application.

It is used by:

-   Android package installation
-   Google Play
-   signing
-   Firebase configuration
-   deep links
-   application updates

Changing it can make Android treat the app as a different application.

------------------------------------------------------------------------

# 11. `versionCode` and `versionName`

Example:

``` text
versionCode = 17
versionName = "1.0.0"
```

`versionName` is human-readable.

`versionCode` is the internal increasing version used for releases.

Think:

``` text
1.0.0 = display version
17    = build/version number
```

------------------------------------------------------------------------

# 12. `AndroidManifest.xml`

Location:

``` text
android/app/src/main/AndroidManifest.xml
```

This describes the Android application to the operating system.

It can define:

-   permissions
-   activities
-   services
-   receivers
-   providers
-   intent filters
-   deep links
-   exported components
-   metadata

Example:

``` xml
<uses-permission android:name="android.permission.CAMERA" />
```

Activity example:

``` xml
<activity
    android:name=".MainActivity"
    ... />
```

When debugging:

``` text
permission issue
deep link issue
activity issue
manifest merger error
service/receiver issue
```

look here.

------------------------------------------------------------------------

# 13. `MainActivity.kt`

The main Android Activity is a native lifecycle/entry point.

Conceptually:

``` text
Android OS
   ↓
MainActivity
   ↓
React Native host
   ↓
React Native runtime
   ↓
JavaScript
```

It is not where you normally write React UI.

Use it for Android-specific behavior such as:

-   Activity lifecycle
-   intents
-   Android-specific launch behavior
-   special native integrations

------------------------------------------------------------------------

# 14. `MainApplication.kt`

This is the Android application-level native entry point.

It participates in initializing the application and React Native host.

It may configure:

-   React Native host
-   Hermes
-   New Architecture
-   native packages
-   application lifecycle
-   native initialization

If an app crashes very early during Android startup, this area can be
relevant.

------------------------------------------------------------------------

# 15. React Native Gradle Plugin

Modern React Native uses the React Native Gradle Plugin.

It handles important build tasks including:

-   React Native Android dependencies
-   JavaScript bundling for release variants
-   Hermes tooling
-   React Native Codegen
-   native module/build configuration
-   Metro integration
-   New Architecture support

A release variant can have tasks similar to:

``` text
createBundleReleaseJsAndAssets
```

This is why a release APK/AAB can contain the JS bundle without Metro
running.

------------------------------------------------------------------------

# 16. Android Debug vs Release

Debug:

``` bash
cd android
./gradlew assembleDebug
```

Typical output:

``` text
android/app/build/outputs/apk/debug/app-debug.apk
```

Release:

``` bash
./gradlew assembleRelease
```

Typical output:

``` text
android/app/build/outputs/apk/release/app-release.apk
```

A release build is optimized and normally contains the production
JavaScript bundle.

------------------------------------------------------------------------

# 17. Android App Bundle

For Google Play:

``` bash
cd android
./gradlew bundleRelease
```

Typical output:

``` text
android/app/build/outputs/bundle/release/app-release.aab
```

Difference:

``` text
APK
= installable Android package

AAB
= publishing bundle used by Google Play
```

Google Play generates device-specific APKs from the AAB.

------------------------------------------------------------------------

# 18. Android build types and variants

Common build types:

``` text
debug
release
```

If you also define flavors:

``` text
staging
production
```

you can get:

``` text
stagingDebug
stagingRelease
productionDebug
productionRelease
```

The Android Gradle Plugin builds variants from:

``` text
product flavor + build type
```

This is useful for different:

-   API URLs
-   application IDs
-   Firebase projects
-   icons
-   feature sets

------------------------------------------------------------------------

# 19. `gradle.properties`

This contains project/Gradle properties.

Depending on the project, you may see:

``` properties
org.gradle.jvmargs=...
android.useAndroidX=true
hermesEnabled=true
newArchEnabled=true
```

It can affect:

-   Gradle memory
-   AndroidX
-   Hermes
-   New Architecture
-   build behavior

When debugging native configuration, inspect it.

------------------------------------------------------------------------

# 20. `local.properties`

Usually contains the local Android SDK path:

``` properties
sdk.dir=/Users/you/Library/Android/sdk
```

This is machine-specific.

Normally do not commit it.

If Android says:

``` text
SDK location not found
```

check this and your Android SDK environment.

------------------------------------------------------------------------

# 21. `proguard-rules.pro`

Release Android builds can use R8/ProGuard.

This file contains custom rules when libraries need special treatment.

If:

``` text
debug works
release crashes
ClassNotFoundException
NoSuchMethodException
reflection failure
```

investigate R8/ProGuard rules.

------------------------------------------------------------------------

# 22. Android `res/`

Android resources live under:

``` text
android/app/src/main/res/
```

Examples:

``` text
drawable/
mipmap-*/
values/
xml/
raw/
```

`mipmap-*` commonly contains launcher icons.

`values/` contains XML values such as:

``` text
strings.xml
colors.xml
themes.xml
```

`drawable/` contains drawable resources.

`xml/` contains various Android configuration files.

------------------------------------------------------------------------

# 23. Android native package flow

Suppose:

``` bash
npm install some-native-package
```

A native package may contain:

``` text
Java/Kotlin
C++
Android resources
Gradle configuration
JavaScript wrapper
```

Modern React Native generally uses autolinking.

Conceptually:

``` text
npm package
   ↓
node_modules
   ↓
React Native autolinking
   ↓
Gradle/native configuration
   ↓
Android compilation
```

Therefore an npm package can affect both JS and native builds.

------------------------------------------------------------------------

# 24. Android build pipeline

When you run:

``` bash
cd android
./gradlew assembleRelease
```

think:

``` text
gradlew
  ↓
Gradle Wrapper
  ↓
Gradle
  ↓
settings.gradle
  ↓
root build configuration
  ↓
app/build.gradle
  ↓
resolve dependencies
  ↓
React Native Gradle Plugin
  ├── compile Kotlin/Java
  ├── compile native libraries
  ├── run Codegen when needed
  ├── bundle JS/assets
  └── invoke Hermes tooling where configured
  ↓
merge manifests
  ↓
process resources
  ↓
link native libraries
  ↓
package APK
  ↓
sign APK
  ↓
app-release.apk
```

------------------------------------------------------------------------

# 25. Android commands to memorize

Build debug:

``` bash
cd android
./gradlew assembleDebug
```

Build release APK:

``` bash
./gradlew assembleRelease
```

Build release AAB:

``` bash
./gradlew bundleRelease
```

Install debug:

``` bash
./gradlew installDebug
```

Clean:

``` bash
./gradlew clean
```

List tasks:

``` bash
./gradlew tasks
```

Detailed error:

``` bash
./gradlew assembleRelease --stacktrace
```

More logging:

``` bash
./gradlew assembleRelease --info
```

------------------------------------------------------------------------

# 26. Install Android without Expo

You can build and install manually:

``` bash
cd android

./gradlew assembleDebug

adb install -r app/build/outputs/apk/debug/app-debug.apk
```

Or:

``` bash
./gradlew installDebug
```

This bypasses Expo CLI/EAS CLI for the build step.

You are still using:

``` text
React Native
Gradle
Android SDK
JDK
Metro/Hermes as required
```

------------------------------------------------------------------------

# 27. Android logs

Check devices:

``` bash
adb devices
```

View logs:

``` bash
adb logcat
```

Clear logs:

``` bash
adb logcat -c
```

Then reproduce the crash.

Look for:

``` text
FATAL EXCEPTION
AndroidRuntime
UnsatisfiedLinkError
ClassNotFoundException
NoSuchMethodError
SecurityException
```

For native Android crashes, `adb logcat` is one of your first tools.

------------------------------------------------------------------------

# 28. iOS folder structure

Typical React Native iOS project:

``` text
ios/
├── RSL Cards.xcodeproj/
│   └── project.pbxproj
├── RSL Cards.xcworkspace/
├── Podfile
├── Podfile.lock
├── RSL Cards/
│   ├── AppDelegate.swift
│   ├── Info.plist
│   ├── Assets.xcassets/
│   ├── LaunchScreen.storyboard
│   └── *.entitlements
└── RSL CardsTests/
```

Older or different React Native projects may use:

``` text
AppDelegate.mm
AppDelegate.h
```

instead of Swift.

Names also vary according to your application name.

------------------------------------------------------------------------

# 29. `.xcodeproj`

Example:

``` text
RSL Cards.xcodeproj
```

This is the Xcode project.

Its internal `project.pbxproj` describes:

-   targets
-   source files
-   resources
-   build phases
-   build settings
-   linked frameworks
-   configurations

Normally let Xcode manage this file instead of manually editing it.

------------------------------------------------------------------------

# 30. `.xcworkspace`

Example:

``` text
RSL Cards.xcworkspace
```

When using CocoaPods, open the workspace:

``` bash
open "RSL Cards.xcworkspace"
```

Why?

The workspace can contain:

``` text
your application project
+
Pods project
+
other projects
```

The `.xcodeproj` alone does not represent the complete CocoaPods
workspace.

------------------------------------------------------------------------

# 31. `Podfile`

The `Podfile` defines iOS native dependencies for CocoaPods.

Conceptually:

``` text
Podfile
   ↓
CocoaPods resolves dependencies
   ↓
Pods
   ↓
Xcode workspace
```

React Native/Expo native modules can be integrated through CocoaPods.

When you have:

``` text
pod install failure
native module missing
framework missing
pod version conflict
```

inspect:

``` text
Podfile
package version
Podfile.lock
```

------------------------------------------------------------------------

# 32. `Podfile.lock`

After:

``` bash
pod install
```

CocoaPods records resolved dependency versions in:

``` text
Podfile.lock
```

This is extremely important.

If:

``` text
developer A works
developer B fails
```

or:

``` text
yesterday worked
today fails after pod install
```

compare the resolved native dependency versions.

For application projects, `Podfile.lock` is normally committed.

------------------------------------------------------------------------

# 33. `Pods/`

CocoaPods places native dependency content here.

Examples:

``` text
Pods/
├── Target Support Files/
├── Headers/
└── installed pods
```

Normally do not manually modify code in `Pods`.

Instead change:

``` text
Podfile
Podfile.lock
native package version
```

and reinstall/update dependencies.

------------------------------------------------------------------------

# 34. `AppDelegate`

`AppDelegate` is a major native iOS entry point.

Depending on project version:

``` text
AppDelegate.swift
```

or:

``` text
AppDelegate.mm
```

Conceptually:

``` text
iOS starts
   ↓
AppDelegate / native application setup
   ↓
React Native initialization
   ↓
JS runtime
   ↓
React application
```

It can participate in:

-   app lifecycle
-   deep links
-   URL handling
-   push notifications
-   native SDK initialization
-   React Native setup

If the app crashes before the React UI appears, inspect native startup
and SDK initialization.

------------------------------------------------------------------------

# 35. `Info.plist`

`Info.plist` contains iOS application configuration.

Examples:

``` text
CFBundleIdentifier
NSCameraUsageDescription
NSLocationWhenInUseUsageDescription
NSPhotoLibraryUsageDescription
CFBundleURLTypes
```

Permissions and URL schemes commonly live here.

When you have:

``` text
camera permission issue
photo permission issue
location permission issue
Google Sign-In redirect issue
deep-link issue
```

inspect `Info.plist`.

------------------------------------------------------------------------

# 36. Bundle Identifier

Example:

``` text
com.rslcards.dealer
```

The bundle identifier connects your application to Apple Developer/App
Store configuration.

It affects:

-   App ID
-   signing
-   provisioning
-   capabilities
-   push notifications
-   keychain access
-   App Store Connect
-   TestFlight

Do not casually change it.

------------------------------------------------------------------------

# 37. Entitlements

Files such as:

``` text
RSL Cards.entitlements
```

define Apple capabilities.

Examples:

``` text
Push Notifications
Associated Domains
App Groups
iCloud
Keychain Sharing
Sign in with Apple
```

If a capability works in development but not in production, check:

``` text
entitlements
Signing & Capabilities
Apple Developer configuration
provisioning
```

------------------------------------------------------------------------

# 38. `Assets.xcassets`

Contains Xcode-managed image assets.

Typical:

``` text
AppIcon.appiconset
```

and other image sets.

------------------------------------------------------------------------

# 39. `LaunchScreen.storyboard`

This is the native launch screen.

It appears before React Native has fully initialized.

Important distinction:

``` text
Launch screen
≠
React Native first screen
```

If you see:

``` text
launch screen
   ↓
app disappears
```

a native startup crash is one possible cause.

------------------------------------------------------------------------

# 40. Xcode targets

A target is something Xcode can build.

You may have:

``` text
RSL Cards
RSL CardsTests
NotificationServiceExtension
ShareExtension
WidgetExtension
```

Each target can have its own:

-   source files
-   resources
-   dependencies
-   signing
-   entitlements
-   build settings

------------------------------------------------------------------------

# 41. Xcode configurations

Common:

``` text
Debug
Release
```

Debug generally has:

``` text
less optimization
debug symbols
debugger
```

Release generally has:

``` text
optimization
production-like settings
different signing/configuration
```

This is a major reason:

``` text
works in Debug
```

does not prove:

``` text
works in TestFlight
```

------------------------------------------------------------------------

# 42. Xcode schemes

A scheme tells Xcode how to:

-   build
-   run
-   test
-   profile
-   archive

A scheme selects things such as:

``` text
target
configuration
environment variables
launch arguments
archive behavior
```

Think:

``` text
Scheme
  ↓
Target
  ↓
Debug/Release
  ↓
Run/Test/Archive settings
```

------------------------------------------------------------------------

# 43. Xcode build phases

Common build phases:

``` text
Compile Sources
Link Binary With Libraries
Copy Bundle Resources
Run Script
```

React Native/native packages can add scripts or dependencies here.

If you see:

``` text
bundle phase failed
script phase failed
framework missing
resource missing
```

inspect Build Phases.

------------------------------------------------------------------------

# 44. Xcode build settings

Build settings control compilation and packaging.

Examples:

``` text
iOS Deployment Target
Swift version
Architectures
Code Signing
Bundle Identifier
Optimization
Header Search Paths
Framework Search Paths
Other Linker Flags
```

When you get:

``` text
framework not found
undefined symbols
module not found
architecture mismatch
signing error
```

inspect target/project build settings.

------------------------------------------------------------------------

# 45. CocoaPods vs Swift Package Manager

iOS dependencies can come from:

``` text
CocoaPods
Swift Package Manager
```

React Native projects commonly use CocoaPods.

If a library uses SPM instead, its dependency may appear in Xcode's
package configuration rather than the Podfile.

First identify which dependency system the library uses.

------------------------------------------------------------------------

# 46. iOS build pipeline

Conceptually:

``` text
xcodebuild
   ↓
read .xcworkspace
   ↓
resolve projects/targets
   ↓
read build settings
   ↓
compile Swift / Objective-C / C++
   ↓
compile React Native native code
   ↓
compile/link Pods
   ↓
bundle JS/assets
   ↓
link frameworks
   ↓
code signing
   ↓
.app
   ↓
archive
   ↓
.ipa
```

------------------------------------------------------------------------

# 47. Build iOS without Expo

Install native dependencies:

``` bash
cd ios
pod install
```

Open workspace:

``` bash
open "RSL Cards.xcworkspace"
```

Then choose:

``` text
iPhone
Debug/Release
```

and Run in Xcode.

Or use `xcodebuild` from Terminal.

------------------------------------------------------------------------

# 48. Find iOS schemes

``` bash
xcodebuild -list -workspace "RSL Cards.xcworkspace"
```

This shows available schemes.

------------------------------------------------------------------------

# 49. Build iOS from Terminal

Example Debug build:

``` bash
xcodebuild \
  -workspace "RSL Cards.xcworkspace" \
  -scheme "RSL Cards" \
  -configuration Debug \
  -sdk iphonesimulator \
  build
```

For a release archive:

``` bash
xcodebuild \
  -workspace "RSL Cards.xcworkspace" \
  -scheme "RSL Cards" \
  -configuration Release \
  -destination "generic/platform=iOS" \
  -archivePath build/RSL-Cards.xcarchive \
  archive
```

The archive is:

``` text
build/RSL-Cards.xcarchive
```

------------------------------------------------------------------------

# 50. `.xcarchive`

An `.xcarchive` contains the archived application plus useful metadata
and symbol files.

Conceptually:

``` text
.xcarchive
├── Products/
├── dSYMs/
└── metadata
```

dSYM files are important for symbolication of release crash reports.

------------------------------------------------------------------------

# 51. Export an IPA

After archiving:

``` bash
xcodebuild -exportArchive \
  -archivePath build/RSL-Cards.xcarchive \
  -exportOptionsPlist ExportOptions.plist \
  -exportPath build/export
```

The export can produce:

``` text
RSL Cards.ipa
```

The export options depend on the distribution method.

------------------------------------------------------------------------

# 52. `.app` vs `.xcarchive` vs `.ipa`

Think:

``` text
.app
= application bundle

.xcarchive
= archived build + symbols + metadata

.ipa
= distributable iOS package
```

Flow:

``` text
Source
 ↓
Build
 ↓
.app
 ↓
Archive
 ↓
.xcarchive
 ↓
Export
 ↓
.ipa
 ↓
TestFlight/App Store
```

------------------------------------------------------------------------

# 53. Simulator vs physical iPhone

Simulator is useful but is not equivalent to a real device.

Physical device adds:

-   real signing
-   real entitlements
-   real APNs
-   real keychain behavior
-   real hardware
-   real camera
-   real device lifecycle
-   real memory constraints

For native production issues:

> Always test a physical device before release.

------------------------------------------------------------------------

# 54. React Native startup

A simplified startup model:

### Android

``` text
Android OS
   ↓
Application / Activity
   ↓
React Native Host
   ↓
JS runtime (often Hermes)
   ↓
Native modules
   ↓
JS bundle
   ↓
React application
```

### iOS

``` text
iOS
 ↓
native application startup
 ↓
AppDelegate / scene lifecycle
 ↓
React Native initialization
 ↓
JS runtime
 ↓
native modules
 ↓
JS bundle
 ↓
React application
```

If the crash occurs before JS starts, a normal React red error may never
appear.

------------------------------------------------------------------------

# 55. Native module architecture

Example: OneSignal.

Conceptually:

``` text
Your TypeScript
      ↓
react-native-onesignal JS API
      ↓
React Native native integration
      ↓
OneSignal native SDK
      ↓
iOS/Android notification APIs
```

Therefore one npm package can affect:

``` text
JavaScript
Gradle
CocoaPods
AndroidManifest
Info.plist
Entitlements
native startup
runtime
```

------------------------------------------------------------------------

# 56. New Architecture

Modern React Native can use:

``` text
JSI
TurboModules
Fabric
Codegen
```

Simplified:

``` text
JavaScript
   ↓
JSI
   ├── TurboModules
   ├── Fabric renderer
   └── native/C++ layer
```

A native library must be compatible with the architecture being used.

This can produce cases such as:

``` text
Debug works
Release crashes
New Architecture crashes
Old Architecture works
```

When this happens, isolate the architecture before changing many
dependencies.

------------------------------------------------------------------------

# 57. Hermes

Hermes is a JavaScript engine commonly used by React Native.

Production concept:

``` text
JS/TS
  ↓
Metro
  ↓
JS bundle
  ↓
Hermes tooling/runtime where configured
  ↓
Native application
```

If errors mention:

``` text
Hermes
bytecode
JS engine
bundle
```

investigate:

``` text
React Native version
Hermes configuration
Metro
release bundling
native build
```

------------------------------------------------------------------------

# 58. Metro

Metro is the React Native JavaScript bundler.

Development:

``` text
App
 ↓
Metro server
 ↓
JavaScript
```

Production:

``` text
Metro
 ↓
bundle JS/assets
 ↓
application package
```

This explains why:

``` text
Debug works
Release fails
```

can happen: Release must successfully bundle the JS and assets into the
native application.

------------------------------------------------------------------------

# 59. Where should you write code?

### Normal application code

Use:

``` text
app/
src/
components/
hooks/
services/
stores/
```

depending on your architecture.

### Android-specific code

Use:

``` text
android/app/src/main/java/...
```

for Kotlin/Java native code.

Examples:

-   Activity behavior
-   Android services
-   receivers
-   custom native modules
-   Android SDK integrations

### iOS-specific code

Use:

``` text
ios/<AppName>/
```

for Swift/Objective-C/Objective-C++.

Examples:

-   AppDelegate behavior
-   native modules
-   iOS SDK integrations
-   extensions
-   lifecycle behavior

Do not put ordinary business logic into `android/` or `ios/`.

------------------------------------------------------------------------

# 60. Where are native packages configured?

A JavaScript-only package may only need:

``` text
package.json
```

A native package can affect:

``` text
package.json
android/settings.gradle
android/app/build.gradle
AndroidManifest.xml
Podfile
Podfile.lock
Info.plist
entitlements
native source
```

Modern React Native autolinking handles much of this automatically.

------------------------------------------------------------------------

# 61. Expo config plugins

Example:

``` json
"plugins": [
  "expo-camera",
  "expo-notifications"
]
```

Conceptually:

``` text
app.json
   ↓
Expo config plugin
   ├── AndroidManifest
   ├── Gradle
   ├── Info.plist
   ├── Podfile
   └── native project configuration
```

This is why changing `app.json` can change native projects.

------------------------------------------------------------------------

# 62. EAS vs local native builds

### Local Android

``` bash
cd android
./gradlew assembleRelease
```

Your Mac runs:

``` text
Gradle
Android SDK
JDK
NDK when needed
Node
React Native build tooling
```

### EAS Android

Conceptually:

``` text
EAS
 ↓
remote build machine
 ↓
Node
 ↓
prebuild if needed
 ↓
Gradle
 ↓
Android SDK/JDK/NDK
 ↓
APK/AAB
```

### Local iOS

``` bash
cd ios
pod install
xcodebuild ...
```

Uses:

``` text
CocoaPods
Xcode
Apple SDK
Node/Metro
React Native
```

### EAS iOS

``` text
EAS
 ↓
macOS build machine
 ↓
Node
 ↓
prebuild if needed
 ↓
CocoaPods
 ↓
Xcode
 ↓
Apple SDK
 ↓
signing
 ↓
.app/.xcarchive/.ipa
```

So EAS is orchestration around native build tooling.

------------------------------------------------------------------------

# 63. What `npx expo run:android` does

Conceptually:

``` text
expo run:android
 ↓
prepare/generate native project if required
 ↓
invoke Android native build
 ↓
Gradle
 ↓
APK
 ↓
install
 ↓
launch
```

It is not a completely separate Android build system.

------------------------------------------------------------------------

# 64. What `npx expo run:ios` does

Conceptually:

``` text
expo run:ios
 ↓
prepare/generate native project if required
 ↓
Pods/native configuration
 ↓
Xcode/xcodebuild
 ↓
.app
 ↓
install
 ↓
launch
```

------------------------------------------------------------------------

# 65. Why EAS can say Build Successful while the app crashes

EAS can successfully complete:

``` text
compile
link
package
sign
upload
```

while the app later fails:

``` text
launch
native initialization
runtime
```

Therefore:

``` text
EAS build = successful
```

does not mean:

``` text
application runtime = verified
```

------------------------------------------------------------------------

# 66. Android native debugging workflow

Build:

``` bash
cd android
./gradlew assembleDebug
```

Install:

``` bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

Logs:

``` bash
adb logcat
```

Clear logs before reproduction:

``` bash
adb logcat -c
```

Then reproduce and search for:

``` text
FATAL EXCEPTION
AndroidRuntime
UnsatisfiedLinkError
ClassNotFoundException
NoSuchMethodError
SecurityException
```

For release-only issues:

``` bash
./gradlew clean
./gradlew assembleRelease
```

and test the release APK on a physical device.

------------------------------------------------------------------------

# 67. iOS native debugging workflow

Open:

``` text
ios/*.xcworkspace
```

Select:

``` text
physical iPhone
```

Run with Xcode.

If it crashes:

``` text
Xcode
 ↓
Debug navigator
 ↓
crashing thread
 ↓
call stack
```

Look for:

``` text
SIGABRT
EXC_BAD_ACCESS
EXC_CRASH
Objective-C exception
Swift fatal error
```

The first useful native frame often identifies the subsystem involved.

------------------------------------------------------------------------

# 68. Common Android errors

### SDK location not found

Check:

``` text
local.properties
ANDROID_HOME
Android SDK installation
```

### Gradle/JDK compatibility

Check:

``` text
JDK
Gradle Wrapper
Android Gradle Plugin
```

### Dependency resolution

Check:

``` text
repositories
Gradle
dependency versions
network
```

### Manifest merger failed

Check:

``` text
AndroidManifest.xml
permissions
activities
providers
metadata
library manifests
```

### Duplicate class

Usually:

``` text
two native libraries
incompatible dependency versions
```

Inspect:

``` bash
./gradlew app:dependencies
```

### AAPT2/resource error

Check:

``` text
res/
AndroidManifest.xml
resource names
SDK versions
```

### UnsatisfiedLinkError

Check:

``` text
ABI
NDK
native .so libraries
packaging
New Architecture
```

------------------------------------------------------------------------

# 69. Common iOS errors

### `No such module`

Check:

``` text
Podfile
pod install
workspace
framework/module integration
```

### `Framework not found`

Check:

``` text
Pods
Link Binary With Libraries
Framework Search Paths
pod installation
```

### `Undefined symbols`

Check:

``` text
linked frameworks
Pods
architecture
Other Linker Flags
native dependency
```

### Code signing error

Check:

``` text
Signing & Capabilities
Team
Bundle Identifier
certificates
profiles
entitlements
```

### Provisioning profile error

Check:

``` text
Apple Developer App ID
capabilities
entitlements
profile
bundle ID
```

### Launch crash

Check:

``` text
AppDelegate
native SDK initialization
Pods
notification SDK
deep links
scene lifecycle
native modules
architecture
OS compatibility
release settings
```

------------------------------------------------------------------------

# 70. The fastest failure-stage decision tree

``` text
Does it compile?
 |
 +-- NO → Gradle / Xcode / Pods / compiler
 |
 YES
 |
 v
Does it install?
 |
 +-- NO → signing / provisioning / packaging / distribution
 |
 YES
 |
 v
Does it launch?
 |
 +-- NO → native startup / SDK / Activity / AppDelegate
 |
 YES
 |
 v
Does JS load?
 |
 +-- NO → Metro / JS bundle / Hermes / release bundling
 |
 YES
 |
 v
Does a feature fail?
 |
 +-- YES → JS or native module
 |
 NO
 |
 v
Works
```

This is one of the most useful mental models for debugging.

------------------------------------------------------------------------

# 71. Release testing for your RSL Cards app

Your project contains native-heavy packages such as:

``` text
expo-camera
expo-notifications
expo-image-picker
expo-secure-store
expo-file-system
Google Sign-In
Apple Authentication
OneSignal
```

Therefore do not use only:

``` bash
npx expo start
```

for final native verification.

Use:

``` text
JavaScript development
        ↓
Native Debug build
        ↓
Native Release build
        ↓
Physical device
        ↓
EAS QA build
        ↓
TestFlight
```

------------------------------------------------------------------------

# 72. Recommended Android release test

``` bash
cd android

./gradlew clean

./gradlew assembleRelease
```

Install:

``` bash
adb install -r app/build/outputs/apk/release/app-release.apk
```

Then test:

``` text
Cold launch
Login
Google Sign-In
Camera
Image picker
Notifications
API
WebSocket/SSE
Navigation
Logout
Background/foreground
Kill and reopen
Device restart
```

------------------------------------------------------------------------

# 73. Recommended iOS release test

Open:

``` text
ios/RSL Cards.xcworkspace
```

Select:

``` text
Physical iPhone
Release
```

Run.

Test the same checklist:

``` text
Cold launch
Login
Google Sign-In
Apple Sign-In
Camera
Photo library
Notifications
API
WebSocket/SSE
Navigation
Logout
Background/foreground
Kill and reopen
Device restart
```

This is much closer to TestFlight behavior than Expo Go.

------------------------------------------------------------------------

# 74. Cold-start testing

Always test:

``` text
Install
 ↓
Open
```

Then:

``` text
Kill app
 ↓
Open
```

Then:

``` text
Restart device
 ↓
Open
```

Then:

``` text
Background
 ↓
wait
 ↓
return
```

Native initialization bugs often appear only during a particular
lifecycle.

------------------------------------------------------------------------

# 75. Clean builds

Android:

``` bash
cd android
./gradlew clean
./gradlew assembleRelease
```

iOS:

``` bash
cd ios
pod install
```

If needed, clean Derived Data through Xcode.

Native build systems cache compiled objects, Pods, Gradle outputs and
other artifacts, so clean builds are useful when results appear
inconsistent.

------------------------------------------------------------------------

# 76. Dependency/version matrix

For native problems, record:

``` text
Node
npm/yarn/pnpm
Expo SDK
React Native
React
Xcode
iOS SDK
CocoaPods
Ruby
JDK
Gradle
Android Gradle Plugin
Android SDK
NDK
Hermes
New Architecture
native package versions
```

A native compatibility problem can exist even when JavaScript code looks
correct.

------------------------------------------------------------------------

# 77. Lock files

JavaScript:

``` text
package-lock.json
yarn.lock
pnpm-lock.yaml
```

iOS:

``` text
Podfile.lock
```

These tell you what was actually resolved.

If:

``` text
developer A works
developer B fails
```

compare the lock files and native dependency versions.

------------------------------------------------------------------------

# 78. How an npm native package reaches the final app

Example:

``` text
npm install react-native-example
        ↓
package.json
        ↓
node_modules/react-native-example
        ↓
JS API
        +
Android native code
        +
iOS native code
        ↓
Autolinking
        ↓
Gradle / CocoaPods
        ↓
Native compilation
        ↓
APK / IPA
```

That is why a package installation can change native build behavior.

------------------------------------------------------------------------

# 79. What happens during `pod install`

Conceptually:

``` text
Podfile
 ↓
CocoaPods dependency resolution
 ↓
download/install native dependencies
 ↓
generate Pods project
 ↓
generate integration files
 ↓
update workspace
 ↓
Podfile.lock
```

Then Xcode should normally open:

``` text
.xcworkspace
```

not only:

``` text
.xcodeproj
```

------------------------------------------------------------------------

# 80. What happens during Gradle dependency resolution

Conceptually:

``` text
settings.gradle
build.gradle
app/build.gradle
version configuration
        ↓
repositories
        ↓
resolve dependencies
        ↓
download/cache artifacts
        ↓
compile/link
```

If dependency resolution fails, the Gradle error normally identifies the
missing artifact/version.

------------------------------------------------------------------------

# 81. Useful Android dependency inspection

Run:

``` bash
cd android
./gradlew app:dependencies
```

For release runtime dependencies:

``` bash
./gradlew app:dependencies --configuration releaseRuntimeClasspath
```

This is useful for finding:

``` text
duplicate dependencies
version conflicts
transitive dependencies
```

------------------------------------------------------------------------

# 82. Useful iOS dependency inspection

Inspect:

``` text
Podfile.lock
```

Search for a library:

``` bash
grep -i "OneSignal" Podfile.lock
```

This tells you the resolved Pod version.

You can also use CocoaPods commands such as:

``` bash
pod outdated
```

when you intentionally want to inspect available updates.

------------------------------------------------------------------------

# 83. Debug vs Release --- why production-only bugs happen

Debug may have:

``` text
Metro server
debug symbols
debugger
less optimization
```

Release may have:

``` text
bundled JS
optimization
Hermes production behavior
R8 on Android
different signing
different native configuration
```

Therefore:

``` text
Debug works
```

does not prove:

``` text
Release works
```

------------------------------------------------------------------------

# 84. Simulator vs physical device

Use both.

``` text
Simulator
= fast development

Physical device
= final native verification
```

Physical-device testing is especially important for:

``` text
camera
push notifications
keychain
signing
entitlements
Apple/Google authentication
native SDKs
device-specific lifecycle
```

------------------------------------------------------------------------

# 85. What to commit

Commonly commit:

``` text
package.json
lock file
app.json/app.config.*
android/
ios/
Podfile
Podfile.lock
native source
Gradle configuration
Xcode project
```

Usually do not commit:

``` text
node_modules/
android/.gradle/
android/app/build/
local.properties
DerivedData
```

For `Pods/`, follow your team's chosen CocoaPods strategy; many
application projects do not commit the generated Pods directory.

------------------------------------------------------------------------

# 86. Native debugging strategy

Do not randomly change five dependencies.

Use:

``` text
1. Identify the failure stage
2. Reproduce locally
3. Determine JS vs native
4. Read native logs
5. Identify the first meaningful native frame
6. Isolate one dependency/configuration
7. Rebuild
8. Compare results
```

Example:

``` text
OneSignal enabled  → crash
OneSignal disabled → works
```

Now you have useful evidence.

Changing OneSignal + React Native + Expo + Xcode simultaneously gives
you much less information.

------------------------------------------------------------------------

# 87. Your current TestFlight problem as an example

There are two fundamentally different cases:

### Install failure

``` text
TestFlight
 ↓
Install
 ↓
❌ requested app unavailable
```

The app did not start.

Investigate:

``` text
App Store Connect
TestFlight
build availability
agreements
distribution
signing/provisioning
Apple/TestFlight service
```

### Runtime crash

``` text
TestFlight
 ↓
Install succeeds
 ↓
Tap app
 ↓
launch
 ↓
💥 crash
```

Investigate:

``` text
native startup
OneSignal
Expo modules
React Native
Xcode/iOS SDK
architecture
release configuration
```

Do not mix these two failure stages.

------------------------------------------------------------------------

# 88. Final architecture diagram

``` text
                         RSL CARDS
                            |
             +--------------+--------------+
             |                             |
             v                             v
       JavaScript/TS                 Native platform
             |                             |
             v                    +--------+--------+
          Metro                   |                 |
             |                    v                 v
             |                 Android             iOS
             |                    |                 |
             |                  Gradle            Xcode
             |                    |                 |
             |              Android SDK        Apple SDK
             |              JDK / NDK          CocoaPods
             |                    |                 |
             |                    v                 v
             |                  APK/AAB            .app
             |                                      |
             |                                      v
             |                                   Archive
             |                                      |
             |                                      v
             |                                     IPA
             |                                      |
             +-------------------+------------------+
                                 |
                                 v
                              Device
```

------------------------------------------------------------------------

# 89. One-page emergency checklist

``` text
[ ] Does it compile?
    NO → Gradle/Xcode/Pods/compiler

[ ] Does it install?
    NO → signing/provisioning/packaging/distribution

[ ] Does it launch?
    NO → native startup/crash

[ ] Does JavaScript load?
    NO → Metro/Hermes/release bundle

[ ] Does one feature fail?
    YES → determine JS vs native module

Android:
[ ] adb devices
[ ] adb logcat
[ ] ./gradlew assembleRelease
[ ] ./gradlew app:dependencies

iOS:
[ ] open .xcworkspace
[ ] physical device
[ ] Release configuration
[ ] Xcode crash stack
[ ] Podfile.lock

Native:
[ ] AndroidManifest.xml
[ ] Info.plist
[ ] entitlements
[ ] Gradle
[ ] Podfile
[ ] native SDK version
[ ] New Architecture
[ ] Hermes
[ ] OS/Xcode/SDK compatibility
```

------------------------------------------------------------------------

# 90. Commands worth memorizing

## Android

``` bash
cd android

./gradlew tasks
./gradlew clean
./gradlew assembleDebug
./gradlew assembleRelease
./gradlew bundleRelease
./gradlew installDebug
./gradlew app:dependencies
./gradlew assembleRelease --stacktrace

adb devices
adb logcat
adb logcat -c
```

## iOS

``` bash
cd ios

pod install
pod outdated

xcodebuild -list -workspace "RSL Cards.xcworkspace"

xcodebuild \
  -workspace "RSL Cards.xcworkspace" \
  -scheme "RSL Cards" \
  -configuration Debug \
  build

xcodebuild \
  -workspace "RSL Cards.xcworkspace" \
  -scheme "RSL Cards" \
  -configuration Release \
  -destination "generic/platform=iOS" \
  -archivePath build/RSL-Cards.xcarchive \
  archive
```

------------------------------------------------------------------------

# 91. The most important files to memorize

## Android

``` text
android/
├── settings.gradle
├── build.gradle
├── gradle.properties
├── gradlew
└── app/
    ├── build.gradle
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/.../
        │   ├── MainActivity.kt
        │   └── MainApplication.kt
        └── res/
```

Mental model:

``` text
settings.gradle
    → project/modules

build.gradle
    → project-level build

app/build.gradle
    → application build

AndroidManifest.xml
    → Android declaration/permissions/components

MainActivity
    → Activity entry/lifecycle

MainApplication
    → application/React Native initialization

res/
    → Android resources
```

## iOS

``` text
ios/
├── RSL Cards.xcodeproj/
├── RSL Cards.xcworkspace/
├── Podfile
├── Podfile.lock
└── RSL Cards/
    ├── AppDelegate.swift
    ├── Info.plist
    ├── *.entitlements
    ├── Assets.xcassets/
    └── LaunchScreen.storyboard
```

Mental model:

``` text
.xcworkspace
    → app + Pods workspace

.xcodeproj
    → Xcode application project

Podfile
    → native dependency definition

Podfile.lock
    → resolved native dependency versions

AppDelegate
    → native application startup/lifecycle

Info.plist
    → iOS application configuration

entitlements
    → Apple capabilities

Assets.xcassets
    → image/icon assets
```

------------------------------------------------------------------------

# 92. Official references

-   Android Gradle build overview:
    https://developer.android.com/build/gradle-build-overview
-   React Native Gradle Plugin:
    https://reactnative.dev/docs/react-native-gradle-plugin
-   React Native Android release builds:
    https://reactnative.dev/docs/signed-apk-android.html
-   Apple Xcode command-line tools:
    https://developer.apple.com/documentation/xcode/xcode-command-line-tool-reference
-   Apple Xcode build system:
    https://developer.apple.com/documentation/xcode/build-system
-   Apple Xcode schemes:
    https://developer.apple.com/documentation/xcode/customizing-the-build-schemes-for-a-project
-   Apple release-build testing:
    https://developer.apple.com/documentation/xcode/testing-a-release-build
-   Apple app distribution/archive:
    https://developer.apple.com/documentation/xcode/distributing-your-app-for-beta-testing-and-releases

------------------------------------------------------------------------

# 93. Final rule

When a native problem happens, first ask:

``` text
WHERE DID IT FAIL?
```

Not:

``` text
WHAT PACKAGE SHOULD I CHANGE?
```

Use this order:

``` text
BUILD
  ↓
INSTALL
  ↓
LAUNCH
  ↓
JS LOAD
  ↓
FEATURE
```

Then choose the correct tool:

``` text
Android build      → Gradle
Android runtime    → adb logcat
iOS build          → Xcode/xcodebuild
iOS runtime        → Xcode crash stack
iOS dependencies   → Podfile/Podfile.lock
JS runtime         → Metro/React Native logs
EAS                → EAS build logs
TestFlight         → App Store Connect crash/distribution data
```

That separation is the key to solving native issues quickly.
