# Expo and EAS: Complete Workflow and Must-Know Concepts

A practical guide to developing, building, testing, publishing, and maintaining React Native applications with Expo and Expo Application Services (EAS).

## Table of Contents

1. [Expo and EAS fundamentals](#1-expo-and-eas-fundamentals)
2. [The complete development-to-production workflow](#2-the-complete-development-to-production-workflow)
3. [Create an Expo application](#3-create-an-expo-application)
4. [Expo Go vs. development builds](#4-expo-go-vs-development-builds)
5. [App configuration and Continuous Native Generation](#5-app-configuration-and-continuous-native-generation)
6. [Connect the project to EAS](#6-connect-the-project-to-eas)
7. [Configure `eas.json` and build profiles](#7-configure-easjson-and-build-profiles)
8. [APK vs. AAB and signing credentials](#8-apk-vs-aab-and-signing-credentials)
9. [Environment variables and secrets](#9-environment-variables-and-secrets)
10. [EAS Update and OTA releases](#10-eas-update-and-ota-releases)
11. [Submit to Google Play and the App Store](#11-submit-to-google-play-and-the-app-store)
12. [CI/CD and automated releases](#12-cicd-and-automated-releases)
13. [Expo for SaaS and white-label applications](#13-expo-for-saas-and-white-label-applications)
14. [Must-know checklist](#14-must-know-checklist)
15. [Common mistakes](#15-common-mistakes)
16. [End-to-end practice workflow](#16-end-to-end-practice-workflow)
17. [Interview questions](#17-interview-questions)
18. [Official documentation](#18-official-documentation)

---

## 1. Expo and EAS fundamentals

### What is React Native?

React Native is a framework for building mobile applications using React and JavaScript or TypeScript. It renders native platform UI components and lets developers share application logic between iOS and Android.

### What is Expo?

Expo is a framework and ecosystem built around React Native. It provides project tooling, APIs, libraries, development workflows, native configuration support, and a consistent way to build cross-platform mobile apps.

### What is EAS?

**Expo Application Services (EAS)** is a collection of hosted services that supports the app lifecycle, including cloud builds, signing-credential management, store submission, over-the-air updates, and build/release automation.

| Tool | Main responsibility |
|---|---|
| React Native | Renders the mobile UI and runs application logic |
| Expo | Framework, APIs, project tooling, and native configuration workflow |
| Expo Go | Prebuilt app for quick development with supported native libraries |
| Development build | Your app's native binary with a development client and required native capabilities |
| EAS Build | Produces Android and iOS application binaries in the cloud |
| EAS Submit | Uploads eligible binaries to app stores |
| EAS Update | Publishes compatible JavaScript and asset updates over the air |
| EAS Workflows | Automates supported build, test, and release processes |

**Important:** Expo does not replace React Native. Expo provides a framework and ecosystem around React Native. EAS is a set of services that can be used with Expo and compatible React Native projects.

---

## 2. The complete development-to-production workflow

A common production workflow looks like this:

1. **Develop:** Build screens, navigation, state management, and API integration using React Native and TypeScript.
2. **Run locally:** Use Expo Go for basic experimentation or a development build for project-specific native capabilities.
3. **Configure:** Set the application name, native identifiers, assets, plugins, environment variables, and build profiles.
4. **Build:** Create development, preview, or production binaries with EAS Build.
5. **Test:** Install and validate the build on devices or emulators; use internal distribution, Google Play testing, or TestFlight as appropriate.
6. **Release:** Submit the binary to the store and complete the platform's release steps.
7. **Maintain:** Monitor issues, fix defects, publish compatible EAS Updates, and create new native builds when native code or configuration changes.

A successful EAS build means a binary has been produced. It does not, by itself, mean the application has been released publicly in an app store.

---

## 3. Create an Expo application

### Prerequisites

For a Windows development environment, prepare:

- Node.js LTS.
- npm, Yarn, or pnpm (use one package manager consistently in each project).
- VS Code or another editor.
- An Android device or Android Emulator.
- Git for source control.
- An Expo account when you are ready to use EAS services.

You can request cloud iOS builds from Windows with EAS. Running the iOS Simulator locally requires macOS.

### Create and start a project

Run in PowerShell:

```powershell
npx create-expo-app@latest MyExpoApp
cd MyExpoApp
npx expo start
```

Open the project in Expo Go or a development build. If a compatible Android Emulator is configured, press `A` in the Expo CLI terminal to open the app on Android.

### Useful development commands

```powershell
# Start the development server
npx expo start

# Clear Metro's cache
npx expo start --clear

# Check project health and common configuration issues
npx expo-doctor

# Check installed packages against Expo SDK compatibility recommendations
npx expo install --check
```

### Install dependencies

Use `npx expo install` for packages with native code so Expo can select a version compatible with your Expo SDK where supported.

```powershell
npx expo install expo-camera
npx expo install expo-notifications
npx expo install expo-dev-client
```

These are examples; install only packages required by your app.

Pure JavaScript dependencies can also be installed with your normal package manager. Keep your package manifest and lockfile consistent across your machine and CI.

---

## 4. Expo Go vs. development builds

### Expo Go

Expo Go is a prebuilt application that includes a known collection of native libraries. It allows you to run JavaScript application code without compiling a custom app binary first.

It is useful for:

- Learning Expo and React Native.
- Rapid UI prototyping.
- Basic navigation, hooks, API calls, and supported Expo APIs.

**Limitation:** You cannot add an arbitrary native module or change Expo Go's native configuration for your project.

### Development builds

A development build is your application's own native binary with `expo-dev-client` and the native dependencies configured for your app.

It is useful for:

- Third-party native modules and SDKs.
- Custom native configuration.
- Push notifications and other native integrations that require project-specific setup.
- Production-oriented development and testing.

You can continue to use Metro, Fast Refresh, and the familiar React Native development workflow.

### When is a new native build required?

| Change | Usually needs a new native build? |
|---|---|
| React component, ordinary JavaScript logic, or styling | No |
| Compatible JavaScript and assets published with EAS Update | No |
| Add a native module that is not in the installed binary | Yes |
| Change native permissions or entitlements | Yes |
| Change native icon or splash configuration | Yes |
| Change native SDK configuration | Usually yes |
| Update a dependency that changes native code | Usually yes |
| Upgrade the Expo SDK or native runtime | Plan for a new build |

**Rule of thumb:** If a change requires native functionality that is not already present in the installed binary, create a new native build.

---

## 5. App configuration and Continuous Native Generation

Expo configuration normally lives in one of these files:

- `app.json`
- `app.config.js`
- `app.config.ts`

Configuration can define application metadata, native identifiers, assets, config plugins, and settings used during native project generation and updates.

### Example `app.json`

This is a simplified example. Replace the sample identifiers with values unique to your application.

```json
{
  "expo": {
    "name": "ShopSphere",
    "slug": "shopsphere",
    "version": "1.0.0",
    "scheme": "shopsphere",
    "icon": "./assets/icon.png",
    "ios": {
      "bundleIdentifier": "com.yourcompany.shopsphere"
    },
    "android": {
      "package": "com.yourcompany.shopsphere"
    },
    "runtimeVersion": {
      "policy": "fingerprint"
    }
  }
}
```

### Important configuration fields

| Property | Purpose |
|---|---|
| `name` | User-facing app name |
| `slug` | Expo project slug |
| `version` | Marketing version, such as `1.0.0` |
| `ios.bundleIdentifier` | iOS app identity |
| `android.package` | Android application ID |
| `icon` | App icon asset |
| `scheme` | Deep-link URL scheme |
| `plugins` | Native configuration plugins |
| `runtimeVersion` | Controls compatibility between installed native builds and OTA updates |

Treat Android package names and iOS bundle identifiers as long-term app identities. Changing them for a later release generally creates a different app identity rather than updating the same store listing.

### What is Continuous Native Generation (CNG)?

Expo can generate the native Android and iOS project directories from app configuration, templates, and config plugins using Prebuild.

```powershell
npx expo prebuild
```

A **config plugin** automates native project configuration—for example, a library might need a specific Android permission or iOS capability.

If you regenerate native directories using `npx expo prebuild --clean`, manual changes that exist only in those generated directories can be lost. For repeatable workflows, prefer supported app configuration and config plugins where practical. Direct native project maintenance is also possible when your project requires it.

---

## 6. Connect the project to EAS

Install EAS CLI and authenticate:

```powershell
npm install --global eas-cli

eas login
eas whoami
```

From the project root, configure EAS Build:

```powershell
eas build:configure
```

This configures the project for EAS Build and creates an `eas.json` file if needed. The project is associated with an Expo project ID as part of setup.

### `npx expo` vs. `eas`

- `npx expo ...` is commonly used for local development, Expo tooling, and package management.
- `eas ...` is used for EAS services such as cloud builds, submissions, credentials, and OTA updates.

They serve different purposes and are commonly used in the same project.

---

## 7. Configure `eas.json` and build profiles

`eas.json` defines EAS build and submission configuration. A **build profile** is a named set of options for a particular kind of build.

### Example `eas.json`

```json
{
  "cli": {
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "channel": "development",
      "environment": "development"
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview",
      "environment": "preview",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "channel": "production",
      "environment": "production",
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {}
  }
}
```

This is a starting point. Add project-specific configuration and verify the settings against the current EAS documentation. If adopting remote build numbering in an existing application, initialize it with the latest build/version numbers already used by the stores.

### Development profile

Purpose: create a development client for developers to test native integration and connect to Metro.

```powershell
eas build --platform android --profile development
```

### Preview profile

Purpose: create an internally distributed build for QA, product owners, or stakeholders.

```powershell
eas build --platform android --profile preview
```

In the example configuration, Android preview builds explicitly use APK format for convenient installation.

### Production profile

Purpose: create a store-oriented binary.

```powershell
eas build --platform android --profile production
eas build --platform ios --profile production
```

To request both supported platforms:

```powershell
eas build --platform all --profile production
```

EAS provides a build details page with logs, status, and downloadable artifacts.

### What does `autoIncrement` do?

When configured and supported by your versioning setup, `autoIncrement: true` automatically advances the platform build number during builds. Use a versioning strategy consistently, especially for apps already published in stores.

---

## 8. APK vs. AAB and signing credentials

### APK vs. AAB

| Format | Typical use |
|---|---|
| `.apk` | Direct installation on a device or emulator; useful for preview testing |
| `.aab` | Google Play distribution; typical EAS production output |
| `.ipa` | iOS app package used in Apple distribution workflows |

An Android App Bundle (`.aab`) is not installed directly in the same way as an APK. Google Play processes the bundle and creates optimized packages for devices.

For public Google Play distribution, the typical production output is an AAB unless your release process has a specific requirement otherwise.

### Signing credentials

Native app distribution requires signing credentials:

- **Android:** a signing key/keystore.
- **iOS:** signing certificates and provisioning profiles.

EAS can manage supported credentials or you can provide your own. Credential tools include:

```powershell
eas credentials
```

Protect your Expo account and credential backups. Losing or mishandling signing credentials can complicate future releases. Follow the current platform-specific instructions for app signing and credential recovery.

---

## 9. Environment variables and secrets

Production apps commonly need different settings for development, preview, and production.

| Environment | Example API URL | Purpose |
|---|---|---|
| Development | `https://dev-api.example.com` | Developer testing |
| Preview | `https://staging-api.example.com` | QA and stakeholder validation |
| Production | `https://api.example.com` | Real users |

### Local environment variable

For example, create a local `.env` file:

```dotenv
EXPO_PUBLIC_API_URL=https://dev-api.example.com
```

Access it in JavaScript or TypeScript:

```typescript
const API_URL = process.env.EXPO_PUBLIC_API_URL;

export async function getProducts() {
  if (!API_URL) {
    throw new Error("API URL is not configured");
  }

  const response = await fetch(`${API_URL}/products`);

  if (!response.ok) {
    throw new Error("Failed to fetch products");
  }

  return response.json();
}
```

For a production application, keep network requests in your API service layer (for example, your existing Axios setup), with consistent error handling and typed responses.

### Configure EAS environments

EAS supports separate `development`, `preview`, and `production` environments. Variables can be configured in the Expo dashboard or with EAS CLI. Confirm the current CLI syntax and variable visibility options against the documentation for your installed EAS CLI version.

Example commands:

```powershell
eas env:set --name EXPO_PUBLIC_API_URL --value https://staging-api.example.com --environment preview --visibility plaintext

eas env:set --name EXPO_PUBLIC_API_URL --value https://api.example.com --environment production --visibility plaintext
```

Make the build profile's `environment` match the desired environment. When publishing an OTA update, select the intended environment explicitly too.

### Security rule: public vs. private values

Anything accessed through an `EXPO_PUBLIC_` variable is bundled into client-side application code and must be considered public.

**Never include these in a mobile client bundle:**

- Database credentials.
- Django `SECRET_KEY`.
- Payment-provider secret keys.
- Private server API keys.
- Any credential that should remain confidential.

A value stored securely in EAS is no longer secret if you embed it into the app bundle. Keep private credentials on your backend. A React Native app backed by Django REST Framework should normally receive a public API URL and user-scoped tokens where needed; sensitive business logic, payment verification, authorization, and database access remain server-side.

---

## 10. EAS Update and OTA releases

EAS Update publishes compatible JavaScript bundles and assets to installed apps without requiring a new store binary for every change.

### Typical OTA workflow

1. Fix a JavaScript or compatible asset issue.
2. Publish an update to a preview channel.
3. Validate the update with a compatible preview build.
4. Publish the approved update to production.
5. Monitor the release and roll back or publish a corrective update if needed.

### Configure EAS Update

Run the configuration command after EAS has been set up:

```powershell
eas update:configure
```

The setup adds the project configuration needed to receive updates.

### Publish to preview

```powershell
eas update --channel preview --message "Fix product filtering" --environment preview
```

### Publish to production

```powershell
eas update --channel production --message "Fix product filtering" --environment production
```

The installed native binary must be configured to receive updates from the matching channel. Specifying `--environment` explicitly makes it clear which environment's variables are used to bundle the update; check current SDK-specific requirements in the official docs.

### Channel vs. branch vs. runtime version

| Concept | Meaning |
|---|---|
| Channel | A label embedded in a build that directs it to an update stream |
| Branch | An ordered sequence of published updates on EAS |
| Runtime version | Identifies which native runtime an update is compatible with |

Channels are mapped to branches. Keep preview and production release flows separate so QA builds do not accidentally receive production releases and production builds do not receive test-only updates.

### Runtime compatibility

Suppose version 1 of an app includes a particular set of native modules. Later, a new release adds a native camera dependency. The older binary does not acquire that native library just because an OTA JavaScript update imports it. A new native build is required.

The `runtimeVersion` mechanism helps prevent an update from being delivered to an incompatible native runtime.

Two commonly used policies are:

- **`appVersion`:** the runtime version follows the app version. Your release process must reliably update the version when native compatibility changes.
- **`fingerprint`:** derives compatibility from inputs that affect the native runtime. This can reduce mistakes from forgetting to update a version, but can result in new builds when native inputs change.

Choose a policy based on your release process and follow the latest Expo guidance.

### What OTA updates cannot do

| Change | OTA update alone sufficient? |
|---|---|
| JavaScript business logic | Usually |
| React UI or styling | Usually |
| Compatible asset changes | Usually |
| Add a native dependency | No |
| Change native permissions or entitlements | No |
| Add a native SDK or change native project configuration | No |
| Replace the native binary | No |

Use OTA updates only for compatible code and assets. Do not treat them as a replacement for native builds or use them to bypass app-store policies.

---

## 11. Submit to Google Play and the App Store

### Android: Google Play

Before submission:

1. Create the app in Google Play Console.
2. Configure the required service account and access for submission.
3. Confirm the application ID, signing setup, version/build numbers, and target testing or release track.
4. Build and test the production binary.

Submit a build:

```powershell
eas submit --platform android
```

Or build and initiate submission together:

```powershell
eas build --platform android --profile production --auto-submit
```

EAS Submit uploads the binary. Store listing details, testing tracks, review requirements, rollout settings, and the final release process still need to be managed.

### iOS: TestFlight and the App Store

App Store distribution requires an appropriately configured bundle identifier and Apple Developer account.

```powershell
eas build --platform ios --profile production
eas submit --platform ios
```

You can run EAS cloud build and submission commands from Windows. After submission, the build is processed in App Store Connect. Public release still requires completion of the applicable Apple review and release steps.

### Internal distribution vs. TestFlight

These are distinct distribution methods:

- **EAS internal distribution:** shares installable test builds with selected testers, subject to platform signing and installation requirements.
- **TestFlight:** distributes iOS beta builds through Apple's App Store Connect workflow.

Choose the right mechanism for the audience and platform.

---

## 12. CI/CD and automated releases

**CI/CD** stands for Continuous Integration and Continuous Delivery/Deployment. For a mobile app, a pipeline commonly runs checks, creates native builds, distributes builds to testers, and submits approved releases.

### Example pipeline

1. Developer pushes code or opens a pull request.
2. Run linting, type checks, and tests.
3. Run Expo dependency/configuration checks.
4. Create a preview build.
5. QA validates the release candidate.
6. An approved workflow creates a production build and submits it or publishes a compatible OTA update.

EAS Workflows can automate supported build and release steps on Expo infrastructure. EAS CLI can also be used from an external CI platform such as GitHub Actions.

### Recommended starting point

Automate validation and preview builds first. Add production publishing only after your testing, credential management, review, and release approval processes are reliable.

---

## 13. Expo for SaaS and white-label applications

If one React Native codebase powers multiple branded apps, separate **runtime branding** from **native app identity**.

| Requirement | Recommended approach |
|---|---|
| Change text, CMS content, theme colors, banners, or feature visibility | Fetch configuration from the backend API |
| Choose supported icons inside app screens | Use remote configuration with safe defaults |
| Change the operating-system launcher icon or native splash screen | Configure native assets and create an appropriate binary |
| Generate separate apps with distinct names and package IDs | Use build-time app configuration and separate build profiles or variants |
| Change native SDKs, permissions, or entitlements | Update native configuration and rebuild |

### Example tenant configuration

A Django backend could return configuration such as:

```json
{
  "tenantId": "tenant-a",
  "brandName": "Tenant A",
  "theme": {
    "primaryColor": "#2357D5",
    "backgroundColor": "#FFFFFF"
  },
  "features": {
    "showLoyalty": true,
    "showChat": false
  },
  "homeSections": ["banner", "categories", "featuredProducts"]
}
```

React Native can use this response to render permitted dynamic UI choices.

However, changing the displayed brand name inside an app is not the same as changing the operating-system app name or launcher icon. Native identity and assets should be treated as build-time concerns, with a repeatable strategy for each branded variant.

---

## 14. Must-know checklist

Use the following checklist to track your learning.

### Expo fundamentals

- [ ] Understand Expo SDK and React Native compatibility.
- [ ] Know the difference between Expo Go and development builds.
- [ ] Use `expo install` and Expo Doctor.
- [ ] Understand Metro and Fast Refresh.

### Native configuration

- [ ] Configure `app.json` or `app.config.ts`.
- [ ] Understand Prebuild, Continuous Native Generation, and config plugins.
- [ ] Identify the changes that require native rebuilds.
- [ ] Configure permissions, icons, splash screens, and app identifiers.

### EAS Build

- [ ] Configure `eas.json` and build profiles.
- [ ] Create development, preview, and production binaries.
- [ ] Understand APK vs. AAB and iOS distribution artifacts.
- [ ] Manage signing credentials and build numbers.

### Release engineering

- [ ] Separate development, preview, and production environments.
- [ ] Protect environment variables and credentials.
- [ ] Understand EAS Submit and store review/release steps.
- [ ] Understand EAS Update, runtime versions, channels, and branches.

### Professional workflow

- [ ] Test store-oriented builds before release.
- [ ] Inspect cloud build logs to debug failed builds.
- [ ] Set up CI/CD with EAS Workflows or another supported CI system.
- [ ] Plan rollbacks and validate native changes.

---

## 15. Common mistakes

### 1. Using Expo Go for every project

Use Expo Go for supported quick experiments. Start using a development build early when native dependencies or custom native configuration are needed.

### 2. Rebuilding for every code change

Ordinary compatible JavaScript and asset changes can be delivered through the development server or EAS Update. Native changes require a new binary.

### 3. Treating OTA as a native rebuild

An OTA update cannot add native modules, alter native permissions, or change the native capabilities of an installed binary.

### 4. Using the wrong profile

- Development: developer workflow and custom development client.
- Preview: testing and internal validation.
- Production: store-oriented release binary.

### 5. Publishing to the wrong channel

Maintain separate preview and production update flows. Check the channel, branch mapping, runtime version, and environment before publishing.

### 6. Exposing credentials in the app

Client-side `EXPO_PUBLIC_` variables are public. Keep confidential API keys, database credentials, and secret configuration on your backend.

### 7. Ignoring build logs

Locate the actual failing build phase before changing configuration. Common phases include dependency installation, app configuration, JavaScript bundling, Gradle, CocoaPods, native compilation, and signing.

### 8. Assuming upload means release

EAS Submit uploads a binary. Store processing, review, testing, rollout, and publication are separate steps.

### 9. Overwriting native customizations during Prebuild

When native folders are generated, changes made only inside them can be lost on regeneration. Prefer supported config plugins or deliberately maintain native projects as part of your build workflow.

---

## 16. End-to-end practice workflow

This sequence creates an Expo app, configures EAS, builds a development client, and then creates preview and production binaries. Complete the credential and store setup before submission.

### Phase A: Create and start the app

```powershell
npx create-expo-app@latest MyExpoApp
cd MyExpoApp
npx expo start
```

### Phase B: Prepare EAS and a development client

```powershell
npx expo install expo-dev-client
npm install --global eas-cli
eas login
eas build:configure
```

### Phase C: Build and run Android development build

```powershell
eas build --platform android --profile development
```

Install the generated development build on the device/emulator. Start Metro for the development client:

```powershell
npx expo start --dev-client
```

### Phase D: Build a preview version

```powershell
eas build --platform android --profile preview
```

Install and test the preview build. Validate API endpoints, authentication, payment flows, deep links, notifications, and any platform-specific behavior that matters to the app.

### Phase E: Build the production binary

```powershell
eas build --platform android --profile production
```

For iOS:

```powershell
eas build --platform ios --profile production
```

### Phase F: Submit to the relevant store

```powershell
eas submit --platform android
```

or:

```powershell
eas submit --platform ios
```

Do this only after configuring store access, signing, identifiers, versioning, and the appropriate production environment. Verify the submitted binary and finish the release process in the applicable store console.

### Phase G: Publish a compatible JavaScript/asset update

After configuring EAS Update and validating the preview channel:

```powershell
eas update --channel preview --message "QA update" --environment preview
```

For an approved production update:

```powershell
eas update --channel production --message "Approved production update" --environment production
```

Only use OTA updates for code and assets compatible with the installed native runtime.

---

## 17. Interview questions

You should be able to explain these concepts without notes:

1. What is the difference between React Native, Expo, Expo Go, and EAS?
2. Why would you choose a development build instead of Expo Go?
3. What is Continuous Native Generation, and why are config plugins useful?
4. What is the difference between `app.json` and `eas.json`?
5. What happens when you run `eas build`?
6. What is the difference between APK, AAB, and IPA artifacts?
7. How do build profiles and EAS environments separate development, preview, and production?
8. How do EAS Update channels and runtime versions work together?
9. Which changes require a new native binary, and which can be delivered as OTA updates?
10. What is the difference between EAS Build and EAS Submit?
11. How do you protect secrets and API configuration in a React Native app?
12. How would you investigate a build that succeeds locally but fails on EAS?
13. What is the purpose of signing credentials?
14. How would you release one codebase as several branded applications?
15. How would you design a safe preview-to-production release process?

---

## 18. Official documentation

Use official Expo documentation as the source of truth because CLI options and SDK behavior can change.

1. [Create an Expo project](https://docs.expo.dev/get-started/create-a-project/)
2. [EAS Build setup](https://docs.expo.dev/build/setup/)
3. [EAS Update introduction](https://docs.expo.dev/eas-update/introduction/)
4. [Configure `eas.json`](https://docs.expo.dev/eas/json/)
5. [EAS environment variables](https://docs.expo.dev/eas/environment-variables/)
6. [Build on CI](https://docs.expo.dev/build/building-on-ci/)
7. [Expo app configuration](https://docs.expo.dev/workflow/configuration/)
8. [Continuous Native Generation](https://docs.expo.dev/workflow/continuous-native-generation/)
9. [EAS Submit](https://docs.expo.dev/submit/introduction/)
10. [EAS credentials](https://docs.expo.dev/app-signing/overview/)

---

## Recommended learning order

1. Master Expo local development and package compatibility.
2. Learn Expo Go vs. development builds and Prebuild/config plugins.
3. Configure EAS and understand build profiles.
4. Learn signing, APK/AAB, and internal distribution.
5. Configure environments and secure client/server configuration.
6. Learn store submission and release management.
7. Learn EAS Update, runtime compatibility, channels, and branches.
8. Finish with CI/CD and white-label build strategies.

For a SaaS-style application, a useful practical project is to create one Expo app with development, preview, and production profiles, add API-driven branding from a Django REST Framework backend, and separately configure native application identities for branded variants.
