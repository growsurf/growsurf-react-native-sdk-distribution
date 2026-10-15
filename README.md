# GrowSurf React Native SDK

Add GrowSurf referral and affiliate programs to a React Native app. The SDK uses the native GrowSurf iOS and Android SDKs and presents the native GrowSurf Window.

React Native 0.74 and newer support both architectures where React Native offers them. Expo requires a development build; Expo Go does not load this native module.

## Install

```sh
npm install growsurf-react-native-sdk
```

For a bare iOS app, run `pod install` in its `ios` directory. Use `pod install --repo-update` if CocoaPods cannot resolve the pinned GrowSurf SDK.

iOS requires iOS 15 or newer. Android requires minimum SDK 24, compile SDK 36, and Kotlin 2.1.21 or newer. Use an Android Gradle Plugin version that supports compile SDK 36.

For Expo, add the plugin to your app configuration, then rebuild the development client:

```json
{
  "expo": {
    "plugins": ["growsurf-react-native-sdk"]
  }
}
```

The plugin adds Android backup and device-transfer exclusions. It keeps your existing rules and merges the GrowSurf exclusions into app-owned XML resources. If the manifest refers to another library's backup XML, copy that XML into your app's `res/xml` directory so the plugin can merge it.

### React Native 0.74 Android setup

React Native 0.74 needs newer Android build tools to compile the native GrowSurf SDK. Use JDK 21, Gradle 8.13, Android Gradle Plugin 8.13.0, Kotlin 2.1.21, and Android build tools 36.0.0. Keep Java and Kotlin bytecode targets at 17.

In `android/build.gradle`, set the existing values and give both build plugins explicit versions:

```groovy
buildscript {
  ext {
    buildToolsVersion = "36.0.0"
    minSdkVersion = 24
    compileSdkVersion = 36
    kotlinVersion = "2.1.21"
    // Keep the app's targetSdkVersion and ndkVersion settings.
  }
  repositories { google(); mavenCentral() }
  dependencies {
    classpath("com.android.tools.build:gradle:8.13.0")
    classpath("com.facebook.react:react-native-gradle-plugin")
    classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion")
  }
}

subprojects { p ->
  p.plugins.withId("org.jetbrains.kotlin.android") {
    p.tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).configureEach {
      kotlinOptions.jvmTarget = "17"
    }
  }
}
```

Set `distributionUrl=https\://services.gradle.org/distributions/gradle-8.13-all.zip` in `android/gradle/wrapper/gradle-wrapper.properties`. Add `react.internal.disableJavaVersionAlignment=true` to `android/gradle.properties`. This bypasses React Native 0.74's Java-alignment helper, which uses a Kotlin extension type that changes in Kotlin 2.1.

Add the Java target and codegen ordering to `android/app/build.gradle`:

```groovy
android {
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_17
    targetCompatibility JavaVersion.VERSION_17
  }
  // Keep the app's other Android settings.
}

if (project.findProperty("newArchEnabled") == "true") {
  tasks.matching { it.name.startsWith("configureCMake") }.configureEach {
    dependsOn(":growsurf-react-native-sdk:generateCodegenArtifactsFromSchema")
  }
}
```

Build with the included compatibility init script:

```sh
cd android
./gradlew assembleRelease \
  --init-script ../node_modules/growsurf-react-native-sdk/android/rn074-gradle-compat.init.gradle \
  --max-workers=2 --no-parallel
```

The script keeps Gradle warnings visible and turns off warnings-as-errors only while compiling React Native 0.74's Gradle plugin. App and SDK compiler checks stay enabled. Set `newArchEnabled=false` or `true` in your app's Gradle properties to select the architecture.

### React Native 0.74 iOS setup

Set your app's iOS deployment target and the `platform :ios` value in its Podfile to `15.0` or newer. Run `pod install --repo-update`.

React Native 0.74 can retain its older `13.4` target for Pods. For a command-line build, include `IPHONEOS_DEPLOYMENT_TARGET=15.0` so the Pods also compile for iOS 15. Replace the workspace and scheme with your app's names:

```sh
cd ios
xcodebuild -workspace YOUR_APP.xcworkspace -scheme YOUR_APP \
  IPHONEOS_DEPLOYMENT_TARGET=15.0 build
```

Keep any higher deployment target that your app or React Native version requires.

## Configure and capture referral links

```ts
import GrowSurf from 'growsurf-react-native-sdk';
import { Linking } from 'react-native';

await GrowSurf.configure({
  campaignId: 'YOUR_CAMPAIGN_ID',
  publicKey: 'YOUR_MOBILE_PUBLIC_KEY',
});

const initialUrl = await Linking.getInitialURL();
if (initialUrl) await GrowSurf.handleDeepLink(initialUrl);

const links = Linking.addEventListener('url', ({ url }) => {
  void GrowSurf.handleDeepLink(url);
});
```

Capture a link before calling `addReferredParticipant`. The native SDK keeps pending attribution and applies it during signup. Pass Branch, AppsFlyer, Adjust or Singular callback parameters to `handleAttributionParameters(parameters, provider)`. `handleDeferredDeepLink()` reads Android's Play Install Referrer; iOS rejects it with `UNSUPPORTED_PLATFORM`.

```ts
const response = await GrowSurf.addReferredParticipant({
  email: 'gavin@hooli.com',
  firstName: 'Gavin',
  lastName: 'Belson',
});

if (response && 'participant' in response) {
  // Use the API's participant result. The SDK stores any participant token it receives.
  const participant = response.participant;
}
```

`addParticipant` creates or finds a participant without requiring a referral. Both methods return the API's JSON. The response can require a participant token instead of returning participant data. `addReferredParticipant` returns `null` when no participant-create request succeeds.

## Open the GrowSurf Window

```ts
const events = GrowSurf.addWindowListener((event) => {
  // Handle open, close, participantCreated, shareTracked, inviteSent,
  // payoutOrSettingsAction and error. Do not log participant tokens.
});

const window = await GrowSurf.presentGrowSurfWindow({
  identity: { type: 'anonymous' },
  theme: { colorScheme: 'system' },
});

// event.windowId identifies the Window and matches window.id.
await window.close();
events.remove();
links.remove();
```

Use `{ type: 'participantFields', input: { email, termsAccepted } }` for signup, or `{ type: 'existingParticipantToken', token }` for a signed-in participant. Issue participant tokens on your backend and pass them to the app. Keep your REST API key on the backend.

The Window renders native controls. Data methods return JSON with generated TypeScript types. `getGrowSurfWindow` returns `null` when the native SDK uses cached or embedded Window data; `forceRefresh: true` bypasses the anonymous cache. Window presentation uses the native typed result and still opens from those local results.

## Data and errors

`GrowSurf` exposes campaign, participant, referral, invite, share, reward, commission, payout, summary, leaderboard, resource, tax and payout-destination methods. The exported types come from the Mobile API contract. Tax collection uses the hosted tax form.

Promise errors expose `code` and `httpStatus`; `httpStatus` is `null` for a local SDK error. Close any open Window and call `shutdown()` when the app's user logs out.

## Bare Android backup rules

Exclude all six SDK preference files from backup and device transfer. Android has one app-wide file for each rule type. Merge these exclusions into any existing rules, including qualified XML resources; keep exclusions required by other SDKs.

Add `android:fullBackupContent="@xml/growsurf_backup_rules"` and `android:dataExtractionRules="@xml/growsurf_data_extraction_rules"` to your manifest's `<application>` element.

Create `res/xml/growsurf_backup_rules.xml`:

```xml
<full-backup-content>
  <exclude domain="sharedpref" path="growsurf_tokens.xml" />
  <exclude domain="sharedpref" path="growsurf_window_cache_encrypted.xml" />
  <exclude domain="sharedpref" path="growsurf_attribution.xml" />
  <exclude domain="sharedpref" path="growsurf_identity.xml" />
  <exclude domain="sharedpref" path="growsurf_window_cache.xml" />
  <exclude domain="sharedpref" path="growsurf_participant_engagement.xml" />
</full-backup-content>
```

Create `res/xml/growsurf_data_extraction_rules.xml`:

```xml
<data-extraction-rules>
  <cloud-backup>
    <exclude domain="sharedpref" path="growsurf_tokens.xml" />
    <exclude domain="sharedpref" path="growsurf_window_cache_encrypted.xml" />
    <exclude domain="sharedpref" path="growsurf_attribution.xml" />
    <exclude domain="sharedpref" path="growsurf_identity.xml" />
    <exclude domain="sharedpref" path="growsurf_window_cache.xml" />
    <exclude domain="sharedpref" path="growsurf_participant_engagement.xml" />
  </cloud-backup>
  <device-transfer>
    <exclude domain="sharedpref" path="growsurf_tokens.xml" />
    <exclude domain="sharedpref" path="growsurf_window_cache_encrypted.xml" />
    <exclude domain="sharedpref" path="growsurf_attribution.xml" />
    <exclude domain="sharedpref" path="growsurf_identity.xml" />
    <exclude domain="sharedpref" path="growsurf_window_cache.xml" />
    <exclude domain="sharedpref" path="growsurf_participant_engagement.xml" />
  </device-transfer>
</data-extraction-rules>
```
