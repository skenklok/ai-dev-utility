# Deployment Security Checklist (Heroku)

Security checks for Heroku deployments, security headers, and infrastructure.

## Table of Contents
1. [Environment & Secrets](#environment--secrets)
2. [HTTPS & Transport Security](#https--transport-security)
3. [Security Headers](#security-headers)
4. [CORS Configuration](#cors-configuration)
5. [Cookie Security](#cookie-security)

---

## Environment & Secrets

### Config Var Security

Check for:
- [ ] Secrets in code instead of Heroku Config Vars
- [ ] `.env` files committed to git
- [ ] `NODE_ENV` not set to 'production'
- [ ] Debug endpoints accessible in production

**Search commands:**
```bash
# Check for secrets in code
grep -rn "password=\|secret=\|apiKey=\|api_key=" --include="*.ts" --include="*.js"

# Check .gitignore
cat .gitignore | grep -E "\.env|\.env\."

# Check for committed .env files
git ls-files | grep "\.env"

# Check for debug routes
grep -rn "debug\|test\|/dev/" --include="*.ts" --include="*.js" | grep -i "route\|app\.\|router\."
```

**Vulnerable patterns:**
```typescript
// ❌ Hardcoded secret
const stripeKey = "sk_live_xxx";

// ❌ Debug endpoint in production
app.get('/debug/users', async (req, res) => {
  const users = await prisma.user.findMany();
  res.json(users);
});
```

**Secure patterns:**
```typescript
// ✅ Environment variable
const stripeKey = process.env.STRIPE_SECRET_KEY;

// ✅ Debug routes behind auth + environment check
if (process.env.NODE_ENV !== 'production') {
  app.get('/debug/users', adminAuth, async (req, res) => {
    // Only accessible in dev
  });
}
```

### Heroku-Specific Checks

Check for:
- [ ] Procfile running debug/dev commands
- [ ] Review apps with production data access

**Check Procfile:**
```bash
# Should not have debug flags
web: node dist/server.js  # ✅ Good
web: npm run dev           # ❌ Dev mode in production
```

---

## HTTPS & Transport Security

### SSL Enforcement

Heroku provides SSL by default on `*.herokuapp.com`, but you must enforce it.

Check for:
- [ ] HTTP requests not redirected to HTTPS
- [ ] Missing `trust proxy` for Express behind Heroku

**Vulnerable patterns:**
```typescript
// ❌ Not enforcing HTTPS
app.listen(3000);

// ❌ Missing trust proxy (headers won't work correctly)
const app = express();
```

**Secure patterns:**
```typescript
// ✅ Trust Heroku's proxy
const app = express();
app.set('trust proxy', 1);

// ✅ Force HTTPS in production
app.use((req, res, next) => {
  if (process.env.NODE_ENV === 'production' && req.headers['x-forwarded-proto'] !== 'https') {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

---

## Security Headers

### Required HTTP Headers

Check for missing or misconfigured headers:

| Header | Purpose | Risk if Missing |
|--------|---------|-----------------|
| `Strict-Transport-Security` | Force HTTPS | Downgrade attacks |
| `Content-Security-Policy` | Prevent XSS | Script injection |
| `X-Frame-Options` | Prevent clickjacking | UI redress attacks |
| `X-Content-Type-Options` | Prevent MIME sniffing | XSS via file upload |
| `Referrer-Policy` | Control referrer info | Information leakage |

**Use helmet middleware:**
```typescript
import helmet from 'helmet';

// ✅ Apply security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.stripe.com"],
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));
```

**Check for missing helmet:**
```bash
# Check if helmet is installed
grep "helmet" package.json

# Find where it's used
grep -rn "helmet" --include="*.ts" --include="*.js"
```

---

## CORS Configuration

### Proper CORS Setup

Check for:
- [ ] CORS allows `*` (any origin)
- [ ] Credentials allowed with wildcard origin
- [ ] Missing origin validation

**Vulnerable patterns:**
```typescript
// ❌ CRITICAL: Allows any origin
app.use(cors());

// ❌ CRITICAL: Wildcard with credentials
app.use(cors({
  origin: '*',
  credentials: true // This combination is blocked by browsers but indicates bad config
}));

// ❌ Reflects any origin (dangerous!)
app.use(cors({
  origin: (origin, callback) => callback(null, true),
  credentials: true
}));
```

**Secure patterns:**
```typescript
// ✅ Explicit allowed origins
const allowedOrigins = [
  'https://myapp.com',
  'https://www.myapp.com',
  process.env.NODE_ENV !== 'production' && 'http://localhost:3000'
].filter(Boolean);

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

**Search commands:**
```bash
# Find CORS configuration
grep -rn "cors\|Access-Control" --include="*.ts" --include="*.js"
```

---

## Cookie Security

### Secure Cookie Flags

Check for:
- [ ] Missing `Secure` flag (cookies sent over HTTP)
- [ ] Missing `HttpOnly` flag (accessible via JavaScript)
- [ ] Missing or weak `SameSite` attribute

**Vulnerable patterns:**
```typescript
// ❌ Insecure cookie
res.cookie('session', token);

// ❌ Missing important flags
res.cookie('session', token, {
  httpOnly: false,  // XSS can steal it
  secure: false,    // Sent over HTTP
  sameSite: 'none'  // CSRF vulnerable without Secure
});
```

**Secure patterns:**
```typescript
// ✅ Secure cookie configuration
res.cookie('session', token, {
  httpOnly: true,              // Not accessible via JS
  secure: process.env.NODE_ENV === 'production', // HTTPS only in prod
  sameSite: 'strict',          // Prevent CSRF
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
  path: '/'
});

// ✅ Express session with secure cookies
import session from 'express-session';

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000
  }
}));
```

**Search commands:**
```bash
# Find cookie usage
grep -rn "\.cookie\|res.cookie\|set-cookie" --include="*.ts" --include="*.js"

# Find session configuration
grep -rn "express-session\|cookie-session" --include="*.ts" --include="*.js"
```

---

## Error Handling

### Information Disclosure

Check for:
- [ ] Stack traces exposed to clients
- [ ] Database errors with connection details
- [ ] Internal paths revealed in errors

**Vulnerable patterns:**
```typescript
// ❌ Exposing stack trace
app.use((err, req, res, next) => {
  res.status(500).json({
    message: err.message,
    stack: err.stack,  // Exposes file paths!
    query: err.query   // Exposes database info!
  });
});
```

**Secure patterns:**
```typescript
// ✅ Generic error for clients, detailed logging
app.use((err, req, res, next) => {
  // Log full details server-side
  console.error({
    message: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method,
    timestamp: new Date().toISOString()
  });

  // Generic message to client
  res.status(500).json({
    error: 'An unexpected error occurred',
    requestId: req.id // For support reference
  });
});
```

---

## Quick Reference Table

| Check | Severity | Files to Inspect |
|-------|----------|------------------|
| Secrets in code | 🔴 Critical | All source files |
| CORS allows * | 🔴 Critical | Server setup |
| Missing helmet | 🟠 High | App initialization |
| HTTP not redirected | 🟠 High | Middleware |
| Insecure cookies | 🟠 High | Cookie/session setup |
| Stack traces exposed | 🟠 High | Error handlers |
| Debug endpoints | 🟡 Medium | Route definitions |
| Missing trust proxy | 🟡 Medium | Express setup |
