# iOS App Store Submission Checklist

Complete checklist for submitting a React Native app to the Apple App Store.

## Table of Contents
1. [Apple Developer Account](#apple-developer-account)
2. [App Store Connect Setup](#app-store-connect-setup)
3. [Build Preparation](#build-preparation)
4. [Metadata & Assets](#metadata--assets)
5. [Submission](#submission)

---

## Apple Developer Account

### Prerequisites

- [ ] Apple ID with two-factor authentication enabled
- [ ] Apple Developer Program membership ($99/year)
- [ ] D-U-N-S Number (if enrolling as organization)
- [ ] Organization legal entity verification (if applicable)

**Verification:**
```bash
# Check your Apple Developer membership status at
# https://developer.apple.com/account
```

---

## App Store Connect Setup

### Create App Record

1. Go to [App Store Connect](https://appstoreconnect.apple.com)
2. Click "My Apps" → "+" → "New App"
3. Fill in required fields:

| Field | Requirements |
|-------|--------------|
| Platform | iOS |
| Name | Max 30 characters, unique on App Store |
| Primary Language | Your default language |
| Bundle ID | Must match Xcode (e.g., `com.yourcompany.appname`) |
| SKU | Internal identifier (any unique string) |

### App Information

- [ ] **Privacy Policy URL** - Required, must be publicly accessible
- [ ] **Category** - Primary category (and optional secondary)
- [ ] **Content Rights** - Declare if using third-party content
- [ ] **Age Rating** - Complete questionnaire

---

## Build Preparation

### Certificates & Provisioning

- [ ] **Distribution Certificate** created in Apple Developer Portal
- [ ] **App ID** registered with correct capabilities
- [ ] **Provisioning Profile** (App Store Distribution) created and downloaded

### Xcode Configuration

```bash
# ios/YourApp.xcodeproj or ios/YourApp.xcworkspace

# Verify in Xcode:
# - Signing & Capabilities → Team selected
# - Signing & Capabilities → Signing Certificate = Distribution
# - General → Version and Build numbers updated
```

### Build Settings Checklist

- [ ] `PRODUCT_BUNDLE_IDENTIFIER` matches App Store Connect
- [ ] Version number incremented (CFBundleShortVersionString)
- [ ] Build number incremented (CFBundleVersion)
- [ ] Debug symbols stripped for release
- [ ] All analytics/crash reporting configured for production

### Create Archive

```bash
# In Xcode:
# 1. Select "Any iOS Device" as build target
# 2. Product → Archive
# 3. Wait for archive to complete
# 4. Window → Organizer → Distribute App
# 5. Select "App Store Connect" → Upload

# Or via command line:
cd ios
xcodebuild -workspace YourApp.xcworkspace \
  -scheme YourApp \
  -configuration Release \
  -archivePath build/YourApp.xcarchive \
  archive

xcodebuild -exportArchive \
  -archivePath build/YourApp.xcarchive \
  -exportPath build \
  -exportOptionsPlist ExportOptions.plist
```

---

## Metadata & Assets

### App Icon

- [ ] 1024x1024 px PNG
- [ ] No transparency, no rounded corners
- [ ] 72 dpi, RGB color space

### Screenshots

Required sizes (at minimum):
- [ ] 6.7" iPhone (1290 x 2796 px) - iPhone 15 Pro Max
- [ ] 6.5" iPhone (1284 x 2778 px) - iPhone 14 Plus
- [ ] 5.5" iPhone (1242 x 2208 px) - iPhone 8 Plus

Optional but recommended:
- [ ] 12.9" iPad Pro (2048 x 2732 px)

**Requirements:**
- 2-10 screenshots per device size
- PNG or JPEG
- No device frames (Apple adds them)
- First 3 screenshots most important (visible before "See More")

### App Preview (Video)

- [ ] Optional but highly recommended
- [ ] 15-30 seconds
- [ ] MP4 or MOV, H.264
- [ ] Matches screenshot dimensions
- [ ] No device frames in video

### Text Content

| Field | Limit | Notes |
|-------|-------|-------|
| App Name | 30 chars | Must be unique on App Store |
| Subtitle | 30 chars | Brief tagline |
| Keywords | 100 chars | Comma-separated, for search |
| Description | 4000 chars | Features and benefits |
| Promotional Text | 170 chars | Can change without update |
| What's New | 4000 chars | Release notes |

### Support Information

- [ ] Support URL (required)
- [ ] Marketing URL (optional)
- [ ] Privacy Policy URL (required)

---

## Submission

### Pre-Submission Checklist

- [ ] App tested on physical devices
- [ ] All placeholder content replaced
- [ ] Crashlytics/error tracking configured
- [ ] Analytics configured
- [ ] Push notification certificates (if using)
- [ ] In-app purchases configured (if using)
- [ ] App thinning/bitcode enabled

### Test Account (If Required)

If app requires login, provide Apple reviewers:
- [ ] Demo account username
- [ ] Demo account password
- [ ] Any additional instructions

### Submit for Review

1. In App Store Connect → Your App → iOS App
2. Select the uploaded build
3. Complete all required metadata
4. Answer export compliance questions
5. Submit for Review

### Review Timeline

- **Typical review time**: 24-48 hours
- **First submission**: May take longer
- **Rejection**: Fix issues and resubmit

### Common Rejection Reasons

| Reason | Prevention |
|--------|------------|
| Crashes | Thorough testing on multiple devices |
| Incomplete metadata | Fill all required fields |
| Placeholder content | Remove all "lorem ipsum" |
| Broken links | Verify all URLs work |
| Login issues | Provide working test credentials |
| Missing privacy policy | Add accessible privacy policy URL |
| In-app purchase issues | Test payment flows thoroughly |

---

## Quick Reference

| Item | Status | Notes |
|------|--------|-------|
| Developer account | ⬜ | |
| App record created | ⬜ | |
| Bundle ID configured | ⬜ | |
| Certificates ready | ⬜ | |
| App icon uploaded | ⬜ | |
| Screenshots uploaded | ⬜ | |
| Description written | ⬜ | |
| Privacy policy URL | ⬜ | |
| Test account ready | ⬜ | |
| Build uploaded | ⬜ | |
| Submitted for review | ⬜ | |
