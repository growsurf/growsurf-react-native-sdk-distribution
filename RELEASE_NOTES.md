# Release notes

## 0.3.3

- Adds optional `groupCode` to attribution: the reward group signup code (`grsf_group`) from a referral link.
- Native dependencies: GrowSurfSDK 0.10.4 and com.growsurf:growsurf-android-sdk:0.10.4.

## 0.3.1

- Updates the native GrowSurf Window to show each earned reward description once and hide rewards that are not approved.
- Native dependencies: GrowSurfSDK 0.10.3 and com.growsurf:growsurf-android-sdk:0.10.3.

## 0.3.0

- `addReferredParticipant` returns `added`, `validReferral`, `referredBy` and `notAddedReason`, like the native SDKs, with the API's participant fields when signup succeeds. It no longer returns `null`.
- `trackShare` accepts only the share types the Mobile API accepts. The `participantCreated` Window event types the participant fields that the native SDKs send.
- Updates the generated Mobile API types. Commission `status` is `PENDING`, `APPROVED`, `PAID` or `REVERSED`. Window share links are `null` when no participant is signed in. Campaign Reward `metadata` is typed.
- Native dependencies: GrowSurfSDK 0.10.2 and com.growsurf:growsurf-android-sdk:0.10.2.

## 0.2.1

- Documents direct GitHub podspec installation for CocoaPods and Expo.
- Native dependencies: GrowSurfSDK 0.10.0 and com.growsurf:growsurf-android-sdk:0.10.0.

## 0.2.0

- Adds the optional referral status filter and starter reward icon fields.
- Updates the native GrowSurf Window with referral status badges, status tooltips, sorting, filtering, and reward icon plates.
- Native dependencies: GrowSurfSDK 0.10.0 and com.growsurf:growsurf-android-sdk:0.10.0.

## 0.1.1

- Reports `MISSING_PARTICIPANT_TOKEN` through the Window error callback on Android when an existing participant needs a backend-issued participant token.
- Native dependencies: GrowSurfSDK 0.9.0 and com.growsurf:growsurf-android-sdk:0.9.1.

## 0.1.0

- Add referral and affiliate programs to bare React Native apps and Expo development builds, with React Native 0.74 and newer.
- Forward native SDK data as raw API JSON with generated TypeScript types.
- Present the native GrowSurf Window and receive its seven callbacks in JavaScript.
- Capture referral links and attribution parameters before participant signup.
- Merge Android backup and device-transfer exclusions through the Expo config plugin.
- Native dependencies: GrowSurfSDK 0.9.0 and com.growsurf:growsurf-android-sdk:0.9.0.
