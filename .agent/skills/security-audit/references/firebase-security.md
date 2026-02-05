# Firebase Security Checklist

Security checks for Firebase Authentication and related services.

## Table of Contents
1. [Firebase Auth Security](#firebase-auth-security)
2. [Admin SDK Security](#admin-sdk-security)
3. [Security Rules](#security-rules)
4. [Token Verification](#token-verification)

---

## Firebase Auth Security

### Client-Side Configuration

Check for:
- [ ] Firebase config exposed with sensitive keys
- [ ] API key restrictions not configured
- [ ] Auth state persistence misconfigured

**Understanding Firebase API Keys:**
Firebase API keys are designed to be public (identify your project), but you must:
- Restrict keys in Google Cloud Console
- Use security rules to protect data
- Never use admin credentials client-side

**Search commands:**
```bash
# Find Firebase config
grep -rn "firebaseConfig\|apiKey.*firebase\|firebase.*apiKey" --include="*.ts" --include="*.js"

# Check for admin SDK in client code (CRITICAL)
grep -rn "firebase-admin\|serviceAccountKey\|FIREBASE_ADMIN" --include="*.ts" --include="*.tsx"
```

**Vulnerable patterns:**
```javascript
// ❌ CRITICAL: Admin SDK in client-side code
import admin from 'firebase-admin';

// ❌ CRITICAL: Service account key in client bundle
const serviceAccount = require('./serviceAccountKey.json');

// ❌ Storing sensitive data based on client auth state only
if (auth.currentUser) {
  grantPremiumAccess(); // No server verification!
}
```

**Secure patterns:**
```javascript
// ✅ Client-side Firebase config (this is OK to be public)
const firebaseConfig = {
  apiKey: "AIza...",        // Public identifier
  authDomain: "app.firebaseapp.com",
  projectId: "my-project"
  // No private keys here
};

// ✅ Admin SDK only on server
// backend/services/firebase-admin.ts
import admin from 'firebase-admin';
admin.initializeApp({
  credential: admin.credential.applicationDefault()
  // Or use GOOGLE_APPLICATION_CREDENTIALS env var
});
```

---

## Admin SDK Security

### Server-Side Firebase Admin

Check for:
- [ ] Service account key committed to git
- [ ] Service account JSON in client bundle
- [ ] Overly permissive service account roles

**Search commands:**
```bash
# Find service account files
find . -name "*serviceAccount*.json" -o -name "*firebase-adminsdk*.json"

# Check .gitignore
cat .gitignore | grep -i "serviceaccount\|firebase.*key"

# Check for hardcoded credentials
grep -rn "private_key\|client_email.*iam.gserviceaccount" --include="*.ts" --include="*.js"
```

**Vulnerable patterns:**
```typescript
// ❌ CRITICAL: Hardcoded service account
const serviceAccount = {
  "type": "service_account",
  "project_id": "...",
  "private_key_id": "...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n..." // Exposed!
};

// ❌ Loading from bundled file
import serviceAccount from '../serviceAccountKey.json';
```

**Secure patterns:**
```typescript
// ✅ Use environment variables
import admin from 'firebase-admin';

if (process.env.FIREBASE_SERVICE_ACCOUNT) {
  const serviceAccount = JSON.parse(process.env.FIREBASE_SERVICE_ACCOUNT);
  admin.initializeApp({
    credential: admin.credential.cert(serviceAccount)
  });
} else {
  // Use Application Default Credentials (recommended for cloud platforms)
  admin.initializeApp({
    credential: admin.credential.applicationDefault()
  });
}

// ✅ Verify .gitignore includes service account files
// .gitignore
// *serviceAccount*.json
// *firebase-adminsdk*.json
```

---

## Security Rules

### Firestore/Realtime Database Rules

If using Firestore or Realtime Database, check for:
- [ ] Rules allow read/write to all (CRITICAL!)
- [ ] Missing user authentication checks
- [ ] Missing data validation in rules

**Vulnerable rules:**
```javascript
// ❌ CRITICAL: Open to everyone
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}

// ❌ Only checks if logged in, not ownership
match /users/{userId} {
  allow read, write: if request.auth != null;
}
```

**Secure rules:**
```javascript
// ✅ Proper user-scoped access
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only access their own data
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Orders belong to user
    match /orders/{orderId} {
      allow read: if request.auth != null && 
                     resource.data.userId == request.auth.uid;
      allow create: if request.auth != null && 
                       request.resource.data.userId == request.auth.uid;
    }
    
    // Admin-only collection
    match /admin/{document=**} {
      allow read, write: if request.auth != null && 
                            request.auth.token.admin == true;
    }
  }
}
```

---

## Token Verification

### Server-Side Token Verification

Check for:
- [ ] ID tokens trusted without server verification
- [ ] Custom claims not validated
- [ ] Token verification missing on protected routes

**Vulnerable patterns:**
```typescript
// ❌ CRITICAL: Trusting client-provided user data
app.post('/api/premium', async (req, res) => {
  const { userId, isPremium } = req.body; // Can be spoofed!
  if (isPremium) {
    grantPremiumAccess(userId);
  }
});

// ❌ Only checking token exists, not verifying
app.use((req, res, next) => {
  if (req.headers.authorization) {
    next(); // Token not actually verified!
  }
});
```

**Secure patterns:**
```typescript
// ✅ Verify Firebase ID token on server
import admin from 'firebase-admin';

const verifyFirebaseToken = async (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing token' });
  }
  
  const idToken = authHeader.split('Bearer ')[1];
  
  try {
    const decodedToken = await admin.auth().verifyIdToken(idToken);
    req.user = {
      uid: decodedToken.uid,
      email: decodedToken.email,
      emailVerified: decodedToken.email_verified,
      // Custom claims
      role: decodedToken.role,
      isPremium: decodedToken.isPremium
    };
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// ✅ Use middleware on protected routes
app.get('/api/profile', verifyFirebaseToken, async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { firebaseUid: req.user.uid }
  });
  res.json(user);
});

// ✅ Check custom claims for premium
app.get('/api/premium-content', verifyFirebaseToken, async (req, res) => {
  if (!req.user.isPremium) {
    return res.status(403).json({ error: 'Premium required' });
  }
  // Serve premium content
});
```

### Custom Claims Security

```typescript
// ✅ Set custom claims server-side only
// This should be triggered by webhook from RevenueCat/Stripe
app.post('/webhooks/subscription', async (req, res) => {
  const { userId, isPremium } = await verifyWebhook(req);
  
  // Set Firebase custom claim
  await admin.auth().setCustomUserClaims(userId, {
    isPremium: isPremium,
    premiumUntil: Date.now() + 30 * 24 * 60 * 60 * 1000
  });
  
  res.json({ success: true });
});
```

---

## Quick Reference Table

| Check | Severity | Files to Inspect |
|-------|----------|------------------|
| Admin SDK in client code | 🔴 Critical | All client-side files |
| Service account committed | 🔴 Critical | .git, config files |
| Firestore rules open | 🔴 Critical | firestore.rules |
| Token not verified server-side | 🔴 Critical | API middleware |
| Custom claims trusted client-side | 🟠 High | Auth flows |
| Missing email verification check | 🟡 Medium | Registration flows |
