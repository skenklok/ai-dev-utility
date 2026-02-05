# Mobile Security Checklist (OWASP MASVS)

React Native and mobile-specific security checks based on OWASP Mobile Top 10 2024.

## Table of Contents
1. [Credential & Secret Storage](#credential--secret-storage)
2. [Code Security](#code-security)
3. [Network Security](#network-security)
4. [Deep Links](#deep-links)
5. [Biometric Authentication](#biometric-authentication)

---

## Credential & Secret Storage

### M1 - Improper Credential Usage (OWASP 2024 #1)

**CRITICAL: Never store secrets in JavaScript bundle**

Check for:
- [ ] Hardcoded API keys in source code
- [ ] Secrets in `.env` files bundled with app
- [ ] Tokens stored in `AsyncStorage` (insecure!)

**Vulnerable patterns:**
```javascript
// ❌ CRITICAL: Hardcoded secret
const API_KEY = "sk_live_abc123...";

// ❌ CRITICAL: AsyncStorage for secrets
await AsyncStorage.setItem('authToken', token);

// ❌ Bundled .env secrets
const apiKey = process.env.STRIPE_SECRET_KEY;
```

**Secure patterns:**
```javascript
// ✅ Use native secure storage
import * as Keychain from 'react-native-keychain';

// Store credentials
await Keychain.setGenericPassword('auth', token);

// Retrieve credentials
const credentials = await Keychain.getGenericPassword();
if (credentials) {
  const token = credentials.password;
}

// ✅ Only PUBLIC keys client-side, secrets on server
const stripePublishableKey = "pk_live_..."; // OK - public key
```

**Search commands:**
```bash
# Find hardcoded secrets
grep -r "sk_live\|sk_test\|api_key\|apiKey\|secret" --include="*.ts" --include="*.js" --include="*.tsx"

# Find AsyncStorage for sensitive data
grep -r "AsyncStorage.setItem.*token\|AsyncStorage.setItem.*password\|AsyncStorage.setItem.*secret"

# Find .env usage
grep -r "process.env\." --include="*.ts" --include="*.js"
```

---

## Code Security

### Source Code Protection

Check for:
- [ ] Source maps enabled in production builds
- [ ] JavaScript obfuscation missing
- [ ] Debug code paths reachable in production
- [ ] ProGuard/R8 not enabled for Android

**Check metro config:**
```javascript
// metro.config.js - source maps should be disabled in production
module.exports = {
  transformer: {
    // ❌ Source maps in production expose code
    sourceMaps: false, // Should be false for production
  },
};
```

**Check Android ProGuard:**
```groovy
// android/app/build.gradle
buildTypes {
  release {
    // ✅ Should be true
    minifyEnabled true
    proguardFiles getDefaultProguardFile('proguard-android.txt')
  }
}
```

**Check for debug code:**
```bash
# Find debug/development code
grep -r "__DEV__\|console.log\|console.debug\|debugger" --include="*.ts" --include="*.tsx"
```

---

## Network Security

### SSL/TLS & Certificate Pinning

Check for:
- [ ] HTTP endpoints used instead of HTTPS
- [ ] SSL certificate pinning not implemented
- [ ] SSL verification disabled

**Vulnerable patterns:**
```javascript
// ❌ HTTP endpoint
fetch('http://api.example.com/data')

// ❌ Disabled SSL verification (React Native)
// This is sometimes done in network config
```

**Secure patterns:**
```javascript
// ✅ HTTPS only
fetch('https://api.example.com/data')

// ✅ Certificate pinning (using react-native-ssl-public-key-pinning or similar)
import { addPinningToFetch } from 'react-native-ssl-public-key-pinning';
```

**Search commands:**
```bash
# Find HTTP endpoints
grep -r "http://" --include="*.ts" --include="*.js" --include="*.tsx" | grep -v "localhost\|127.0.0.1"

# Find fetch/axios calls
grep -r "fetch(\|axios\." --include="*.ts" --include="*.js"
```

---

## Deep Links

### Deep Link Injection Prevention

Check for:
- [ ] Deep link parameters not validated before use
- [ ] Sensitive actions triggered via deep links without re-auth
- [ ] URL scheme hijacking protection

**Vulnerable patterns:**
```javascript
// ❌ Unvalidated deep link parameters
const { userId } = useParams();
await loadUserProfile(userId); // Direct use without validation

// ❌ Sensitive action without re-authentication
Linking.addEventListener('url', (event) => {
  if (event.url.includes('/reset-password')) {
    resetPassword(extractToken(event.url)); // No verification!
  }
});
```

**Secure patterns:**
```javascript
// ✅ Validate deep link parameters
const { userId } = useParams();
if (!isValidUUID(userId)) {
  return <ErrorScreen />;
}

// ✅ Re-authenticate for sensitive actions
Linking.addEventListener('url', async (event) => {
  if (event.url.includes('/reset-password')) {
    const isValidToken = await verifyTokenServer(extractToken(event.url));
    if (!isValidToken) return;
    // Proceed with additional user verification
  }
});
```

**Files to check:**
- `app.json` / `app.config.js` - scheme configuration
- Navigation configuration files
- Deep link handlers

---

## Biometric Authentication

### Secure Biometric Implementation

Check for:
- [ ] Biometric result trusted without server verification
- [ ] Fallback to less secure method
- [ ] Biometric data stored in app (should never happen)

**Vulnerable patterns:**
```javascript
// ❌ Client-only biometric check
const result = await LocalAuthentication.authenticateAsync();
if (result.success) {
  grantAccess(); // No server verification!
}
```

**Secure patterns:**
```javascript
// ✅ Biometric unlocks secure storage, which has server-verified token
const result = await LocalAuthentication.authenticateAsync();
if (result.success) {
  const token = await SecureStore.getItemAsync('authToken');
  const verified = await verifyTokenWithServer(token);
  if (verified) grantAccess();
}
```

---

## Quick Reference Table

| Check | Severity | Files to Inspect |
|-------|----------|------------------|
| Hardcoded secrets | 🔴 Critical | All `.ts`, `.js`, `.tsx` files |
| AsyncStorage for tokens | 🔴 Critical | Auth-related files |
| Source maps in prod | 🟠 High | `metro.config.js`, build configs |
| ProGuard disabled | 🟠 High | `android/app/build.gradle` |
| HTTP endpoints | 🟠 High | API files, config files |
| No cert pinning | 🟡 Medium | Network configuration |
| Unvalidated deep links | 🟠 High | Navigation, linking handlers |
| Client-only biometrics | 🟠 High | Auth screens, biometric handlers |
