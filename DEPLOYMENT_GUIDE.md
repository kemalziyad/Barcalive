# Barcalive Deployment Guide

This guide covers setting up, building, and deploying the Barcalive Android application.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Local Development Setup](#local-development-setup)
3. [Building the Application](#building-the-application)
4. [Release Signing Configuration](#release-signing-configuration)
5. [Google Play Store Deployment](#google-play-store-deployment)
6. [CI/CD Pipeline Setup](#cicd-pipeline-setup)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software
- **Android Studio** 2024.1 or later ([Download](https://developer.android.com/studio))
- **JDK 11 or higher** (included with Android Studio)
- **Android SDK 36** (API level 36)
- **Minimum SDK support:** API 24 (Android 7.0)
- **Git** for version control

### API Keys & Services
- **Gemini API Key** (for AI features) - [Get from Google AI Studio](https://ai.google.dev/)
- **Stripe Secret Key** (for payments) - [Get from Stripe Dashboard](https://dashboard.stripe.com/)
- **Google Play Console Account** - [Sign up here](https://play.google.com/console)

### Hardware Requirements
- Modern CPU (quad-core or better)
- Minimum 16GB RAM
- 50GB free disk space (for SDK, emulator, builds)
- Android device or emulator for testing

---

## Local Development Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/kemalziyad/Barcalive.git
cd Barcalive
git checkout barcaApp
```

### Step 2: Extract Project Files

```bash
# If using the zip file
unzip barcalive-social-simulator.zip

# Or if files are already extracted, navigate to project root
cd Barcalive
```

### Step 3: Configure Environment Variables

**Create `.env` file** with your API keys:

```bash
# Copy the example file
cp .env.example .env

# Edit .env with your actual keys
nano .env
```

**Contents of `.env`:**
```env
GEMINI_API_KEY=your_actual_gemini_api_key_here
STRIPE_SECRET_KEY=sk_test_your_actual_stripe_key
```

⚠️ **IMPORTANT:** Never commit `.env` to version control. It's already in `.gitignore`.

### Step 4: Configure Local Properties

```bash
# Copy the example local.properties
cp local.properties.example local.properties

# Edit for your setup (optional for debug builds)
nano local.properties
```

### Step 5: Open in Android Studio

1. Launch Android Studio
2. Select **File > Open**
3. Navigate to the `Barcalive` directory
4. Click **Open**
5. Wait for Gradle to sync (first sync may take 5-10 minutes)

### Step 6: Verify Setup

```bash
# Test build configuration
./gradlew assembleDebug --dry-run

# Check for lint issues
./gradlew lint
```

---

## Building the Application

### Debug Build (for development/testing)

```bash
# Build debug APK
./gradlew assembleDebug

# Output: app/build/outputs/apk/debug/app-debug.apk
```

**Install on connected device:**
```bash
./gradlew installDebug
```

**Run on emulator/device:**
```bash
./gradlew runDebug
```

### Release Build (for distribution)

```bash
# Build release APK (requires signing config)
./gradlew assembleRelease

# Output: app/build/outputs/apk/release/app-release.apk
```

### App Bundle (recommended for Play Store)

```bash
# Build App Bundle
./gradlew bundleRelease

# Output: app/build/outputs/bundle/release/app-release.aab
```

### Build with Specific Configuration

```bash
# Clean build (removes old artifacts)
./gradlew clean assembleRelease

# Build with verbose output
./gradlew assembleRelease --info

# Build with optimization
./gradlew assembleRelease --stacktrace
```

### Build Outputs

| Build Type | Output Path | Use Case |
|-----------|-------------|----------|
| Debug APK | `app/build/outputs/apk/debug/app-debug.apk` | Testing on device |
| Release APK | `app/build/outputs/apk/release/app-release.apk` | Direct distribution |
| App Bundle | `app/build/outputs/bundle/release/app-release.aab` | Google Play Store |

---

## Release Signing Configuration

### Understanding Release Signing

Release signing ensures:
- ✅ Your app is authentic and untampered
- ✅ Users can verify app authenticity
- ✅ Play Store tracks your app versions
- ✅ Automatic updates are secure

### Generate a Keystore

If you don't have a keystore, create one:

```bash
keytool -genkey -v -keystore my-upload-key.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload \
  -keypass your_key_password \
  -storepass your_store_password
```

**Interactive prompts:**
```
What is your first and last name?
> [Your Name]
What is the name of your organizational unit?
> Development
What is the name of your organization?
> Barcalive
What is the name of your City or Locality?
> [Your City]
What is the name of your State or Province?
> [Your State]
What is the two-letter country code for this unit?
> US
```

### Configure Signing in Gradle

**Option 1: Environment Variables (Recommended)**

```bash
export KEYSTORE_PATH="/path/to/my-upload-key.jks"
export STORE_PASSWORD="your_store_password"
export KEY_PASSWORD="your_key_password"

# Then build
./gradlew assembleRelease
```

**Option 2: local.properties (for development only)**

Edit `local.properties`:
```properties
KEYSTORE_PATH=/Users/yourname/my-upload-key.jks
STORE_PASSWORD=your_store_password
KEY_PASSWORD=your_key_password
```

Then build:
```bash
./gradlew assembleRelease
```

### Verify Signing Certificate

```bash
# Check certificate details
keytool -list -v -keystore my-upload-key.jks

# Get certificate fingerprint
keytool -list -v -keystore my-upload-key.jks | grep -A 2 "SHA1"
```

---

## Google Play Store Deployment

### Step 1: Prepare Your Google Play Console Account

1. Go to [Google Play Console](https://play.google.com/console)
2. Create a new app or select existing
3. Fill in app details:
   - **App name:** Barcalive
   - **Package name:** com.aistudio.barcalive.hkmqpa (from build.gradle.kts)
   - **App type:** App
   - **Category:** Social / Entertainment
4. Accept agreements

### Step 2: Generate Service Account Key

For CI/CD automation:

1. Go to **Settings > API access**
2. Click **Create new service account**
3. Follow Google Cloud Console link
4. Create service account with Play Console role
5. Download JSON key file
6. Store securely (use GitHub Secrets, never commit)

### Step 3: Build App Bundle

```bash
# Ensure version is updated in build.gradle.kts
# versionCode = 2 (increment from previous)
# versionName = "1.1" 

./gradlew bundleRelease
```

### Step 4: Upload to Play Store

**Manual Upload:**
1. Go to Google Play Console
2. Navigate to **Release > Production** (or test track first)
3. Click **Create new release**
4. Upload `app/build/outputs/bundle/release/app-release.aab`
5. Add release notes
6. Review and submit

**Automated Upload (using fastlane):**

```bash
# Install fastlane
sudo gem install fastlane

# Initialize fastlane
fastlane init android

# Upload release
fastlane android upload_to_play_store track:internal
```

### Step 5: Track Selection (Recommended Approach)

**For first release:** Internal Testing
- Limited to max 100 internal testers
- No review time
- Quick feedback

**After validation:** Closed Testing (Alpha/Beta)
- Up to 1,000 testers for Alpha
- Up to 10,000 testers for Beta
- 2-4 hour review time

**After testing:** Production
- Available to all users
- 24-48 hour review time

### Step 6: Monitor Rollout

After submission, monitor:
- **Crash rates** in Analytics
- **User reviews** on store listing
- **Performance metrics** in Vitals
- **Error logs** in Crashes & ANRs

---

## CI/CD Pipeline Setup

### GitHub Actions Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Android CI/CD

on:
  push:
    branches: [ barcaApp ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ barcaApp ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: gradle
    
    - name: Create .env file
      run: |
        echo "GEMINI_API_KEY=${{ secrets.GEMINI_API_KEY }}" > .env
        echo "STRIPE_SECRET_KEY=${{ secrets.STRIPE_SECRET_KEY }}" >> .env
    
    - name: Make gradlew executable
      run: chmod +x ./gradlew
    
    - name: Run lint checks
      run: ./gradlew lint
    
    - name: Run unit tests
      run: ./gradlew test
    
    - name: Build debug APK
      run: ./gradlew assembleDebug
    
    - name: Build release APK (on tag)
      if: startsWith(github.ref, 'refs/tags/')
      run: |
        echo "KEYSTORE_PATH=${{ secrets.KEYSTORE_PATH }}" >> $GITHUB_ENV
        echo "STORE_PASSWORD=${{ secrets.STORE_PASSWORD }}" >> $GITHUB_ENV
        echo "KEY_PASSWORD=${{ secrets.KEY_PASSWORD }}" >> $GITHUB_ENV
        ./gradlew assembleRelease
    
    - name: Build app bundle
      if: startsWith(github.ref, 'refs/tags/')
      run: ./gradlew bundleRelease
    
    - name: Upload to Play Store (Internal Testing)
      if: startsWith(github.ref, 'refs/tags/')
      uses: r0adkll/upload-google-play@v1
      with:
        serviceAccountJsonPlainText: ${{ secrets.PLAY_CONSOLE_SERVICE_ACCOUNT }}
        packageName: com.aistudio.barcalive.hkmqpa
        releaseFiles: 'app/build/outputs/bundle/release/app-release.aab'
        track: internal
        status: completed
    
    - name: Upload artifacts
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: apks
        path: app/build/outputs/apk/

```

### Configure GitHub Secrets

Add these to **Settings > Secrets and variables > Actions**:

```
GEMINI_API_KEY: your_actual_key
STRIPE_SECRET_KEY: sk_test_your_key
KEYSTORE_PATH: base64_encoded_keystore_file
STORE_PASSWORD: your_store_password
KEY_PASSWORD: your_key_password
PLAY_CONSOLE_SERVICE_ACCOUNT: {"type": "service_account", ...}
```

**To encode keystore for secrets:**
```bash
base64 -i my-upload-key.jks -o keystore.b64
cat keystore.b64
# Copy output to GitHub Secrets as KEYSTORE_PATH
```

### Trigger Automated Deployment

Tag a commit to trigger release build:
```bash
git tag v1.1
git push origin v1.1
```

GitHub Actions will automatically:
1. Build release APK/bundle
2. Run tests
3. Upload to Play Store (internal testing)

---

## Troubleshooting

### Common Build Issues

#### 1. Gradle Sync Fails
```bash
# Clear gradle cache
rm -rf ~/.gradle/caches

# Retry sync
./gradlew clean build
```

#### 2. Permission Denied on gradlew
```bash
chmod +x ./gradlew
```

#### 3. Missing Dependencies
```bash
# Update dependencies
./gradlew --refresh-dependencies clean build
```

#### 4. Memory Issues During Build
Edit `gradle.properties`:
```properties
org.gradle.jvmargs=-Xmx8192m
```

#### 5. Build Completes but App Won't Install
```bash
# Check target device/emulator API level
adb shell getprop ro.build.version.sdk

# Must be API 24 or higher for this app
```

### Signing Issues

#### Certificate Expired
```bash
# Check expiration
keytool -list -v -keystore my-upload-key.jks

# Re-upload APK with same certificate
# (Must use same keystore for Play Store updates)
```

#### Wrong Package Name
Verify in `build.gradle.kts`:
```kotlin
applicationId = "com.aistudio.barcalive.hkmqpa"
```

### Play Store Upload Issues

#### "This APK is signed with the wrong certificate"
Use the same keystore from your Play Console account.

#### "Version code X has already been used"
Increment `versionCode` in `build.gradle.kts`:
```kotlin
versionCode = 2  // Must be higher than previous
versionName = "1.1"
```

#### "Insufficient rights to perform this operation"
Ensure your Google Play Console service account has **Play Console User** or **Release Manager** role.

### Runtime Issues

#### App Crashes on Launch
Check logs:
```bash
adb logcat | grep "com.aistudio.barcalive"
```

#### Database Migration Errors
App uses Room database version 2. Check `AppDatabase.kt`:
```kotlin
@Database(
    entities = [...],
    version = 2,
    exportSchema = false
)
```

#### API Key Not Recognized
Verify `.env` file exists and contains valid keys:
```bash
cat .env
# Should show: GEMINI_API_KEY=your_key
```

---

## Support & Resources

- **Android Documentation:** https://developer.android.com/docs
- **Google Play Console Help:** https://support.google.com/googleplay/android-developer
- **Gradle Documentation:** https://gradle.org/documentation
- **Kotlin Documentation:** https://kotlinlang.org/docs
- **Compose Documentation:** https://developer.android.com/compose

---

## Quick Reference

```bash
# Full build and deploy flow
./gradlew clean                    # Clean previous builds
./gradlew lint                     # Check code style
./gradlew test                     # Run unit tests
./gradlew assembleDebug            # Build debug APK
./gradlew bundleRelease            # Build release bundle
./gradlew installDebug             # Install on device
./gradlew uninstallDebug           # Remove from device
```

---

**Last Updated:** 2026-05-23  
**Version:** 1.0  
**Status:** Production Ready
