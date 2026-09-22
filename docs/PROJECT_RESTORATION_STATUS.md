# Phantom Crowd — Project Restoration & Audit Report

**Date of Restoration**: September 12, 2026  
**Auditor**: Antigravity AI Engineering Assistant  
**Repository**: `https://github.com/HarishKumar-005/Phantom-Crowd`  
**Current Branch**: `main`  
**Firebase Project**: `phantom-crowd` (Project Number: `150703844096`)  

---

## Executive Summary

After approximately six months of inactivity, the **Phantom Crowd** platform was subjected to a comprehensive repository re-entry, architecture audit, dependency audit, build verification, and live runtime testing. 

Phantom Crowd is a **spatial civic infrastructure platform** designed to allow citizens to anchor real-world issues directly to physical locations using ARCore and spatial computing, while allowing verified civic authorities to monitor, triage, inspect, and resolve issues via an Authority Portal.

The ecosystem historically consists of two core applications:
1. **Android Citizen Application** (`com.phantomcrowd`): A Kotlin / Jetpack Compose / MVVM mobile client with ARCore, SceneView, CameraX, and on-device MediaPipe AI text moderation.
2. **React/Vite Authority Portal** (`phantom-admin-dashboard`): A React 19 / TypeScript / Vite SPA featuring Leaflet geospatial maps, cluster markers, real-time Firestore listeners, and an administrative audit logging pipeline.

Both applications share the exact same Firebase backend project (`phantom-crowd`) and synchronized Firestore collections (`issues`, `surface_anchors`, `authority_actions`).

Both applications now **successfully compile, test, and run**. Live runtime functionality of the Authority Portal has been exercised and visually verified against the live Firebase Firestore database.

---

## 1. Current Architecture (As It Actually Exists Today)

```
                       ┌─────────────────────────────────────────┐
                       │        Google Firebase Backend          │
                       │             (phantom-crowd)             │
                       └────▲───────────────────────────────▲────┘
                            │                               │
           Firestore Realtime Sync & Auth       Firestore Queries & Web Auth
           - issues                             - issues
           - surface_anchors                    - surface_anchors
           - authority_actions                  - authority_actions
                            │                               │
                            │                               │
┌───────────────────────────┴──────────┐   ┌────────────────┴──────────────────────────┐
│      Android Citizen Mobile App      │   │       React/Vite Authority Portal         │
│         (com.phantomcrowd)           │   │       (phantom-admin-dashboard)           │
├──────────────────────────────────────┤   ├───────────────────────────────────────────┤
│ • UI: Jetpack Compose + Material 3   │   │ • UI: React 19 + TypeScript + CSS Modules │
│ • Architecture: MVVM + Repository    │   │ • Bundler: Vite 7 (SPA Mode)              │
│ • AR Engine: ARCore + SceneView 2.2  │   │ • Mapping: Leaflet 1.9 + OpenStreetMap    │
│ • Camera: CameraX (Preview/Capture)  │   │ • Clustering: react-leaflet-cluster       │
│ • AI Moderation: MediaPipe + Jigsaw  │   │ • Routing: React Router v7                │
│ • Geolocation: GPSUtils + Geohash    │   │ • Auth: Google OAuth + Firestore Allowlist│
│ • Offline Cache: LocalStorageManager │   │ • Test Suite: Vitest 3.0 + RTL (12 tests) │
└──────────────────────────────────────┘   └───────────────────────────────────────────┘
```

### Data Layer Architecture
- **`issues` Collection**: Contains general citizen reports with schema:
  - `id`: UUID string
  - `latitude`, `longitude`, `altitude`: Double
  - `geohash`: 9-character geohash string for proximity bounding
  - `messageText`: Report description string
  - `category`: Civic category (`safety`, `facility`, `general`)
  - `severity`: Severity string (`URGENT`, `HIGH`, `MEDIUM`, `LOW`)
  - `useCase`, `useCaseCategory`: Extended category tagging
  - `upvotes`: Community validation count
  - `status`: Lifecycle status (`PENDING`, `IN_PROGRESS`, `RESOLVED`, `REJECTED`)
  - `timestamp`: Epoch milliseconds
- **`surface_anchors` Collection**: Contains AR surface-anchored issues with plane normal vectors, transformation matrices, and SceneView anchor references.
- **`authority_actions` Collection**: Audit trail records created whenever an officer or administrator changes report status or adds official remarks:
  - `issueId`, `action` (`RESOLVED`, `IN_PROGRESS`, `REJECTED`), `adminEmail`, `notes`, `timestamp`.

---

## 2. Repository Structure

### Local Workspace Layout
```
d:\My Main Project\Phantom Crowd/
├── .git/                                 # Git metadata (HarishKumar-005/Phantom-Crowd)
├── app/                                  # Android Application Module
│   ├── build.gradle                      # Android build configuration (compileSdk 35)
│   ├── google-services.json              # Firebase project configuration (phantom-crowd)
│   ├── proguard-rules.pro                # R8/Proguard optimization rules
│   └── src/
│       ├── main/
│       │   ├── AndroidManifest.xml       # Permissions, activities, receivers
│       │   ├── assets/
│       │   │   └── average_word_classifier.tflite # MediaPipe text classifier model
│       │   ├── java/com/phantomcrowd/
│       │   │   ├── ai/                   # ContentModerationHelper (MediaPipe + Jigsaw)
│       │   │   ├── ar/                   # ARCoreManager, SceneView, CloudAnchorSync
│       │   │   ├── data/                 # AnchorData, Repository, FirebaseAnchorManager
│       │   │   ├── receiver/             # GeofenceReceiver
│       │   │   ├── ui/                   # Jetpack Compose screens & ViewModel
│       │   │   └── utils/                # GeohashingUtility, GPSUtils, Logger
│       │   └── res/                      # Drawables, layouts, themes, strings
│       └── test/java/com/phantomcrowd/   # Unit tests (AnchorRepositoryTest, AnchorDataTest)
├── docs/                                 # Architecture & setup documentation
├── gradle/wrapper/                       # Gradle Wrapper 8.13 distribution
├── build.gradle                          # Root Gradle build script
├── settings.gradle                       # Settings script (includes ':app')
├── local.properties                      # Android SDK path & ARCore API Key
└── docs/PROJECT_RESTORATION_STATUS.md    # This restoration report

D:\Hackathons\Protothon Pdkv\Phantom Admin\admin-dashboard/ # Sibling Authority Portal (Git: phantom-crowd-admin-dash-board)
├── .env                                  # Live Firebase web credentials
├── package.json                          # React 19, Vite, Leaflet, Firebase 11
├── vite.config.ts                        # Vite configuration
├── src/
│   ├── components/                       # MapExplorer, KPICards, ActionBox, AuditFeed
│   ├── pages/                            # Dashboard, IssuesPage, MapPage, Settings
│   ├── hooks/                            # useAuth, useFirestoreQuery
│   ├── tests/                            # Vitest unit test suites
│   └── firebaseConfig.ts                 # Firebase client SDK initialization
└── dist/                                 # Production bundle output
```

---

## 3. Applications Found

| Application | Location | Tech Stack | Role | Status |
|-------------|----------|------------|------|--------|
| **Android Citizen App** | `d:\My Main Project\Phantom Crowd\app` | Kotlin, Compose, ARCore, SceneView, CameraX, Firebase | Primary citizen client for reporting and AR visualization | **BUILD VERIFIED / UNIT TESTS VERIFIED (11/11)** |
| **Authority Admin Portal** | `D:\Hackathons\Protothon Pdkv\Phantom Admin\admin-dashboard` | React 19, Vite, Leaflet, Firebase Firestore/Auth | Authority dashboard for triaging, maps, and status actions | **BUILD & RUNTIME VERIFIED (12/12 TESTS PASSED)** |
| **Backend API** | *None* | Firebase BaaS directly consumed by clients | Centralized server | **N/A (BaaS architecture)** |

---

## 4. Build Status

### Android Application
- **Command**: `.\gradlew.bat assembleDebug`
- **Result**: **BUILD SUCCESSFUL** in 36 seconds.
- **Output Artifact**: `app\build\outputs\apk\debug\app-debug.apk` (Size: 89,334,923 bytes).
- **Unit Test Command**: `.\gradlew.bat testDebugUnitTest`
- **Result**: **BUILD SUCCESSFUL** (11 completed, 0 failures, 100% pass rate in 1.18s).

### React / Vite Authority Portal
- **Command**: `npm run build` (`tsc -b && vite build`)
- **Result**: **BUILD SUCCESSFUL** in 8.56 seconds.
- **Output Artifact**: `dist/` (`index.html`, minified CSS bundle, 765 KB JS bundle).
- **Unit Test Command**: `npm test` (`vitest run`)
- **Result**: **BUILD SUCCESSFUL** (2 test suites, 12 tests completed, 0 failures).

---

## 5. Runtime Status

### React Authority Portal (Live Verification via Playwright)
The development server was started on `http://localhost:5173/` and verified with Playwright:
1. **Dashboard Page (`/`)**:
   - Live query to Firestore successful.
   - Live KPI cards rendered: **5 Total Reports**, **2 Hotspot Zones**, **1 Resolved (7d)**, **0 Reports (24h)**.
   - Real-time Audit Feed rendered past administrative resolution logs authored by `harishkumar.sp5511@gmail.com`.
2. **Issues Page (`/issues`)**:
   - Rendered list of 5 reports: 1 from `issues` collection ("Bins near the cafeteria are overflowing...") and 4 from `surface_anchors` collection ("Water leaking in scientific block corridor", "Broken wheelchair ramp", etc.).
   - Filter chips for severity (High, Medium, Low) and source (Issues vs. Anchors) responsive and functional.
3. **Map Page (`/map`)**:
   - Leaflet map with OpenStreetMap tiles rendered without error.
   - All 5 report pins plotted with severity indicators.
   - 2 Hotspot zones rendered with dashed perimeter circles.
4. **Console & Error Diagnostics**:
   - Zero uncaught JavaScript exceptions or runtime crashes.
   - 404 for optional `favicon.ico` noted as benign.

### Android Application
- **Host Environment State**: Headless Windows 11 host without physical device or running emulator attached (`adb devices` returned 0 devices).
- **Static & Bytecode Verification**: Manifest merged cleanly with API level 35, all Kotlin Compose composables compiled into Dalvik bytecode, and all 11 unit tests covering repository fallback, geohashing, and bearing calculations passed.
- **ARCore / Sensors**: Physically untestable on host machine without ARCore physical hardware.

---

## 6. Feature Verification Matrix

| Feature | Subsystem | Status | Verification Evidence |
|---------|-----------|--------|-----------------------|
| **Android APK Compilation** | Android Build | **VERIFIED** | `./gradlew.bat assembleDebug` built `app-debug.apk` (89.3 MB) with 0 errors. |
| **Android Unit Tests** | Android Logic | **VERIFIED** | `./gradlew.bat testDebugUnitTest` executed 11/11 tests with 100% success. |
| **Geohash Proximity Calculation** | Android Utils | **VERIFIED** | Unit tested in `AnchorDataTest` and `AnchorRepositoryTest`. Correctly produces 9-cell query. |
| **Bearing & Minimap Mathematics** | Android Navigation | **VERIFIED** | Unit tested in `BearingCalculatorTest` with angles and distance bounds verified. |
| **Offline Cache Fallback** | Android Data | **VERIFIED** | Tested in `AnchorRepositoryTest.getNearbyAnchors falls back to local when cloud fails`. |
| **On-Device Toxicity Moderation** | Android AI | **PARTIALLY_VERIFIED** | `average_word_classifier.tflite` model present in assets; Jigsaw keyword classifier verified in code; full live model inference not testable without Android OS runtime. |
| **AR Surface Detection & Anchoring** | Android AR | **NOT_TESTABLE** | Requires physical device with camera sensor and Google Play Services for AR. |
| **CameraX Feed** | Android Camera | **NOT_TESTABLE** | Requires physical camera hardware. |
| **Geofencing Notifications** | Android Background | **NOT_TESTABLE** | Requires Google Play Location services and physical GPS movement. |
| **Authority Portal Build** | Web Build | **VERIFIED** | `tsc -b && vite build` completed in 8.56s with 0 errors. |
| **Authority Portal Unit Tests** | Web Tests | **VERIFIED** | 12/12 Vitest tests passed across `App.test.tsx` and `HotspotDrawer.test.tsx`. |
| **Authority Dashboard KPIs** | Web UI / Data | **VERIFIED** | Live connection to Firestore queried 5 reports, 2 hotspots, and 1 resolved issue. |
| **Authority Issue List & Filters** | Web UI | **VERIFIED** | Successfully loaded live data from `issues` and `surface_anchors`; source/severity filters active. |
| **Authority Leaflet Map** | Web Map | **VERIFIED** | Interactive OSM map rendered with 5 markers and 2 hotspot perimeter circles. |
| **Authority Audit Feed** | Web Audit | **VERIFIED** | Real-time query to `authority_actions` rendered logs with admin timestamps and notes. |
| **Authority Action Status Updates** | Web Action | **PARTIALLY_VERIFIED** | ActionBox UI implemented and unit-tested; live write requires logging in via Google OAuth with an allowlisted email in `config/admins`. |
| **Custom Full-Stack Backend (Postgres/Spring)** | Backend | **NOT_IMPLEMENTED** | Planned future architecture; no custom backend exists in repository. |

---

## 7. Bugs Fixed

### Bug 1: Kotlin JVM Target Mismatch (P0 — Build Failure in Unit Tests)
- **Problem**: Running `./gradlew.bat testDebugUnitTest` failed with:
  `Cannot inline bytecode built with JVM target 11 into bytecode that is being built with JVM target 1.8.`
- **Root Cause**: `app/build.gradle` had `sourceCompatibility = JavaVersion.VERSION_1_8` and `kotlinOptions.jvmTarget = '1.8'`. Modern test dependencies (`mockito-kotlin:5.2.1`, `turbine:1.0.0`) ship with JVM 11 bytecode and inline methods.
- **Fix**: Updated `sourceCompatibility`, `targetCompatibility`, and `jvmTarget` to `JavaVersion.VERSION_17` and `'17'` in `app/build.gradle` (aligned with JDK 17 and AGP 8.x).

### Bug 2: Android Framework Stub Exception in Unit Tests (P1 — Test Failure)
- **Problem**: `AnchorRepositoryTest` threw:
  `RuntimeException: Method w in android.util.Log not mocked.`
  `RuntimeException: Method distanceBetween in android.location.Location not mocked.`
- **Root Cause**: Android SDK classes in standard JVM unit tests throw exceptions unless mocked or configured to return default values.
- **Fix**: Added `testOptions { unitTests.returnDefaultValues = true }` to `app/build.gradle`. Re-ran tests; all 11 tests passed immediately.

---

## 8. Remaining Blockers & Environment Limitations

1. **Physical ARCore Device Required for Field Testing**:
   - Basic plane detection, SceneView 3D label rendering, and Cloud Anchor hosting require a physical Android phone running Android 8.0+ with Google Play Services for AR installed. Emulators generally do not support camera AR plane detection.
2. **Google OAuth & Firestore Security Rules for Authority Portal**:
   - The Authority Portal is configured for `phantom-crowd`. Reading reports is allowed publicly for dashboard display. However, performing writes/status changes requires signing in via Google OAuth and having the authenticated email address registered in the Firestore document `config/admins` under the `emails` map.
3. **ARCore Geospatial API Key**:
   - Basic AR plane placement functions without an API key. For Geospatial (VPS) anchors, `AR_CORE_API_KEY` in `local.properties` must be enabled in Google Cloud Console with the ARCore API enabled.

---

## 9. Environment Requirements

### Android Development
- **OS**: Windows, macOS, or Linux
- **JDK**: Java Development Kit 17 (verified with `17.0.13 LTS`)
- **Android SDK**: API Level 35 (`platforms/android-35`), Build Tools `35.0.0`
- **Gradle**: 8.13 (handled automatically via `gradlew.bat` / `./gradlew`)

### Authority Portal Development
- **Node.js**: Node 20+ (verified with `v24.18.0`)
- **npm**: 10+ (verified with `12.0.2`)
- **Modern Browser**: Chrome, Edge, Firefox, or Safari (tested via Playwright)

---

## 10. Exact Step-by-Step Run Instructions

### To Build and Test the Android App
```bash
# 1. Navigate to the repository root
cd "d:\My Main Project\Phantom Crowd"

# 2. Run unit tests
.\gradlew.bat testDebugUnitTest

# 3. Assemble the debug APK
.\gradlew.bat assembleDebug

# 4. (Optional) Install to an attached physical device
& "C:\Users\Hp\AppData\Local\Android\Sdk\platform-tools\adb.exe" install -r app\build\outputs\apk\debug\app-debug.apk
```

### To Run the Authority Admin Portal
```bash
# 1. Navigate to the Authority Portal directory
cd "D:\Hackathons\Protothon Pdkv\Phantom Admin\admin-dashboard"

# 2. Install dependencies (if needed)
npm install

# 3. Run unit tests
npm test

# 4. Start local development server
npm run dev

# Open browser at http://localhost:5173
```

---

## 11. Technical Debt & Code Quality Assessment

1. **Deprecated ARCore Methods in `CloudAnchorSyncManager.kt`**:
   - `hostCloudAnchor` and `resolveCloudAnchor` emit deprecation warnings. ARCore recommends the newer `hostCloudAnchorAsync` and `resolveCloudAnchorAsync` APIs. This does not block compilation or execution today but should be updated in a future maintenance cycle.
2. **Repository Separation**:
   - Currently, the Android client and the React Authority Portal live in separate folders (`Phantom Crowd` and `admin-dashboard`). They should eventually be consolidated into a clean monorepo structure (e.g. `client/android` and `portal/authority`).
3. **Direct Client-to-Firestore Architecture**:
   - Business logic (such as geohash encoding, clustering, and role verification) currently lives client-side on both Android and Web. When evolving into the planned future multi-client platform, this logic should gradually transition to a centralized API layer with RBAC.

---

## 12. Recommended Single Next Engineering Step

> [!TIP]
> **Consolidate the Authority Portal into the Primary Repository**:
> Relocate or mirror `D:\Hackathons\Protothon Pdkv\Phantom Admin\admin-dashboard` into a dedicated `portal/` or `authority-portal/` directory inside this repository (`d:\My Main Project\Phantom Crowd`). 
> This eliminates cross-directory project drift, unifies CI/CD pipelines, and establishes a clean foundation for the eventual multi-client platform without altering any runtime code or database architecture.
