# Android Library Test Project - TKA-9308

## Purpose
This test project reproduces the Android Gradle Plugin version incompatibility issue reported in TKA-9308.

## Issue Being Tested
**Error from customer logs:**
```
Build file '/tmp/ws-scm/vista-rn-front-end/modules/serve-module/android/build.gradle' line: 25
* What went wrong:
A problem occurred evaluating root project 'android'.
> org/gradle/initialization/BuildCompletionListener
```

## Root Cause
- **Customer's Android Gradle Plugin:** 3.4.1 (released 2019, requires Gradle 5.1-5.6.4)
- **Mend Scanner Gradle Version:** 8.14.3 (released 2024)
- **Compatibility:** Incompatible - Plugin too old for scanner's Gradle version

## Project Structure
```
android-library-test/
├── build.gradle              # Customer's build file with Android Gradle Plugin 3.4.1
├── settings.gradle           # Gradle project settings
├── gradle.properties         # Android configuration
├── package.json              # NPM dependencies for React Native Gradle plugin
├── src/
│   └── main/
│       └── AndroidManifest.xml  # Required for Android library
└── README.md                 # This file
```

## Key Files

### build.gradle
- **Line 25:** `apply plugin: 'com.android.library'` - The problematic line
- **Line 18:** `classpath ("com.android.tools.build:gradle:3.4.1")` - Old plugin version
- **Dependencies:** React Native (`com.facebook.react:react-native:+`)

### package.json (CRITICAL for fix validation)
**Added:** 2025-12-10

This file is **required** for the React Native Gradle plugin fix to work:
```json
{
  "dependencies": {
    "@react-native/gradle-plugin": "^0.74.0",
    "react-native": "^0.74.0"
  }
}
```

**Why it's needed:**
1. The fix detects React Native by finding `settings.gradle` files containing "node_modules"
2. It walks up directories (maxParentDirLevels: 3→10) looking for `package.json`
3. When found, it runs `npm install` to create `node_modules/@react-native/gradle-plugin`
4. Without `package.json`, the fix cannot work and node_modules remains missing

**Testing the fix:**
- ✅ With package.json: Scanner runs `npm install`, creates node_modules, Gradle can resolve plugin
- ❌ Without package.json: Scanner cannot install dependencies, node_modules missing, test fails

## Expected Results When Scanned

### Expected Errors:
1. **Version Incompatibility:**
   ```
   A problem occurred evaluating root project 'android'.
   > org/gradle/initialization/BuildCompletionListener
   ```

2. **SDK Not Found (if Android SDK not configured):**
   ```
   SDK location not found. Define location with sdk.dir in the local.properties file
   or with an ANDROID_HOME environment variable.
   ```

3. **Deprecated Repository Warning:**
   ```
   WARNING: The jcenter repository is deprecated
   ```

### Expected Scan Result:
- ❌ Gradle dependency resolution fails
- ❌ 0 dependencies reported
- ❌ partialScanError: "gradle"

## How to Use This Test Project

### Option 1: Upload to GitHub (Recommended)
```bash
cd /Users/palinasupranovich/PycharmProjects/TKA_tickets/TKA-9308/android-library-test
git init
git add .
git commit -m "Test case for Android Gradle Plugin version incompatibility - TKA-9308"
# Create new GitHub repo and push
```

### Option 2: Add to Existing Test Repo
```bash
# Copy this directory to your gradle-TKA-9308 repo
cp -r /Users/palinasupranovich/PycharmProjects/TKA_tickets/TKA-9308/android-library-test \
      /path/to/gradle-TKA-9308/android-library-test
cd /path/to/gradle-TKA-9308
git add android-library-test/
git commit -m "Add Android Gradle Plugin version test"
git push
```

### Option 3: Test Locally with Mend CLI
```bash
cd /Users/palinasupranovich/PycharmProjects/TKA_tickets/TKA-9308/android-library-test
mend dependencies --dir . --format json
# Check output for Gradle resolution errors
```

## Validation Checklist

After scanning, verify:
- [ ] Scan attempted to resolve Gradle dependencies
- [ ] Error message mentions line 25 or BuildCompletionListener
- [ ] 0 dependencies reported for Gradle
- [ ] Logs show Android Gradle Plugin 3.4.1 incompatibility

## Related Issues

This test reproduces **Issue #2** from TKA-9308:
- **Issue #1:** Missing node_modules (React Native Gradle plugin) - Already reproduced
- **Issue #2:** Android Gradle Plugin version incompatibility - This test

Both issues contribute to 0 dependencies being reported for the customer.

## Customer Information
- **Ticket:** TKA-9308
- **Customer:** Vista Group International Limited
- **Original File:** `/modules/serve-module/android/build.gradle`
- **Bug Report:** `/Users/palinasupranovich/PycharmProjects/TKA_tickets/TKA-9308/bug-report/bug-report-draft.md`

---

**Created:** 2025-10-30
**Purpose:** QA reproduction testing for TKA-9308