# Google Play Store Submission Checklist

Complete checklist for submitting a React Native app to Google Play Store.

## Table of Contents
1. [Google Developer Account](#google-developer-account)
2. [Play Console Setup](#play-console-setup)
3. [Build Preparation](#build-preparation)
4. [Store Listing](#store-listing)
5. [Testing Requirements](#testing-requirements)
6. [Submission](#submission)

---

## Google Developer Account

### Prerequisites

- [ ] Google Account
- [ ] Google Play Developer registration ($25 one-time fee)
- [ ] Identity verification completed (required for new accounts)

**Note for new accounts (after November 2023):**
New personal developer accounts must complete closed testing with 20+ testers for 14 consecutive days before publishing publicly.

---

## Play Console Setup

### Create App

1. Go to [Google Play Console](https://play.google.com/console)
2. Click "Create app"
3. Fill in required fields:

| Field | Requirements |
|-------|--------------|
| App name | Max 30 characters |
| Default language | Your primary language |
| App or game | Select appropriate type |
| Free or paid | Cannot change after publish |

### App Content Setup

Complete all sections in "App content":

- [ ] **Privacy policy** - Required URL
- [ ] **App access** - Login requirements for reviewers
- [ ] **Ads** - Declare if app contains ads
- [ ] **Content rating** - Complete IARC questionnaire
- [ ] **Target audience** - Age groups (affects review)
- [ ] **News apps** - Declare if news app
- [ ] **COVID-19 apps** - Declaration if applicable
- [ ] **Data safety** - Data collection and sharing practices
- [ ] **Government apps** - Declaration if applicable
- [ ] **Financial features** - If offering financial services

---

## Build Preparation

### Keystore Setup

```bash
# Generate release keystore (do this ONCE and save securely!)
keytool -genkeypair -v \
  -keystore my-release-key.keystore \
  -alias my-key-alias \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000

# ⚠️ CRITICAL: Back up your keystore file and passwords!
# You CANNOT update your app without the same keystore!
```

### Configure Gradle

```groovy
// android/app/build.gradle

android {
    defaultConfig {
        applicationId "com.yourcompany.appname"
        versionCode 1        // Increment for each upload
        versionName "1.0.0"  // User-visible version
    }
    
    signingConfigs {
        release {
            storeFile file('my-release-key.keystore')
            storePassword System.getenv("KEYSTORE_PASSWORD")
            keyAlias 'my-key-alias'
            keyPassword System.getenv("KEY_PASSWORD")
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

### Build App Bundle (AAB)

```bash
cd android

# Clean and build release
./gradlew clean

# Build App Bundle (preferred over APK)
./gradlew bundleRelease

# Output: android/app/build/outputs/bundle/release/app-release.aab
```

### Build Checklist

- [ ] Version code incremented from previous release
- [ ] Release keystore used (not debug)
- [ ] ProGuard/R8 enabled for release
- [ ] All debug logging removed
- [ ] Correct applicationId set
- [ ] Target SDK meets requirements (API 34 for 2024)

---

## Store Listing

### App Details

| Field | Limit | Notes |
|-------|-------|-------|
| App name | 30 chars | Unique, no keyword spam |
| Short description | 80 chars | Below screenshots |
| Full description | 4000 chars | Features, benefits |

### Graphics Assets

| Asset | Dimensions | Format |
|-------|------------|--------|
| App icon | 512x512 px | PNG, 32-bit |
| Feature graphic | 1024x500 px | PNG or JPEG |
| Screenshots | 320-3840 px | PNG or JPEG (24-bit) |

### Screenshot Requirements

- [ ] Minimum 2 screenshots
- [ ] Maximum 8 per device type
- [ ] Aspect ratio 16:9 or 9:16
- [ ] Phone screenshots required
- [ ] Tablet screenshots (recommended for tablet apps)

### Promo Video (Optional)

- [ ] YouTube URL only
- [ ] Keep under 30 seconds
- [ ] Feature USP in first 10 seconds
- [ ] Design for muted autoplay

---

## Testing Requirements

### Closed Testing (Required for New Accounts)

If your account was created after November 13, 2023:

1. **Create closed test track**
   - Play Console → Testing → Closed testing
   - Create new track → Name it (e.g., "Friends & Family Beta")

2. **Add testers**
   - Minimum 20 testers required
   - Add by email or Google Group
   - Testers must opt-in via link

3. **Testing period**
   - 14 consecutive days minimum
   - Testers must actively use the app
   - Track must remain active during period

4. **Verification**
   - Play Console will show testing progress
   - Cannot proceed to production until requirement met

### Internal Testing (Optional, Quick)

- Up to 100 testers
- No review required
- Available immediately after upload
- Good for quick QA cycles

---

## Submission

### Pre-Submission Checklist

- [ ] App tested on multiple Android versions
- [ ] App tested on different screen sizes
- [ ] No placeholder content
- [ ] All required store listing fields completed
- [ ] Privacy policy URL accessible
- [ ] Content rating completed
- [ ] Data safety section completed
- [ ] App access instructions provided (if login required)

### Production Release

1. **Upload AAB**
   - Production → Create new release
   - Upload app-release.aab

2. **Add Release Notes**
   - What's new in this version
   - Max 500 characters per language

3. **Rollout Strategy**
   | Option | Use Case |
   |--------|----------|
   | 100% rollout | Confident release |
   | Staged rollout (1-99%) | Gradual, monitor issues |
   | Specific countries | Geo-targeted launch |

4. **Submit for Review**

### Review Timeline

- **New apps**: 3-7 days (can be longer)
- **Updates**: 1-3 days typically
- **Rejection**: Fix and resubmit

### Common Rejection Reasons

| Reason | Prevention |
|--------|------------|
| Policy violation | Read Developer Program Policies |
| Broken functionality | Test thoroughly |
| Misleading metadata | Accurate descriptions |
| Insufficient testers | Ensure 20+ active testers (new accounts) |
| Privacy issues | Complete data safety accurately |
| Intellectual property | Don't use others' trademarks |

---

## Staged Rollout Best Practices

### Recommended Rollout Schedule

| Day | Percentage | Action |
|-----|------------|--------|
| 1 | 1% | Monitor crash rates |
| 2 | 5% | Check user feedback |
| 3 | 10% | Verify analytics working |
| 4 | 25% | Check revenue (if applicable) |
| 5 | 50% | Major milestone check |
| 6 | 100% | Full release |

### Halt Conditions

Stop rollout immediately if:
- Crash-free rate drops below 99%
- ANR rate exceeds 0.5%
- Significant negative reviews
- Critical bug reports

```
Play Console → Release → Halt rollout
```

---

## Quick Reference

| Item | Status | Notes |
|------|--------|-------|
| Developer account | ⬜ | |
| Identity verified | ⬜ | |
| App created | ⬜ | |
| Keystore created & backed up | ⬜ | |
| AAB built and signed | ⬜ | |
| Store listing complete | ⬜ | |
| App icon uploaded | ⬜ | |
| Feature graphic uploaded | ⬜ | |
| Screenshots uploaded | ⬜ | |
| Content rating done | ⬜ | |
| Data safety done | ⬜ | |
| Privacy policy URL | ⬜ | |
| Closed testing complete (if required) | ⬜ | |
| Submitted for review | ⬜ | |
