# API Security Checklist

Security checks for authentication, authorization, rate limiting, and input validation.

## Table of Contents
1. [JWT Security](#jwt-security)
2. [Authorization Middleware](#authorization-middleware)
3. [Rate Limiting](#rate-limiting)
4. [Input Validation](#input-validation)
5. [Error Responses](#error-responses)

---

## JWT Security

### Token Configuration

Check for:
- [ ] Algorithm not specified (vulnerable to "none" attack)
- [ ] Weak secret (should be 256+ bits)
- [ ] Missing expiration
- [ ] No refresh token rotation
- [ ] Tokens not invalidated on logout

**Search commands:**
```bash
# Find JWT usage
grep -rn "jsonwebtoken\|jwt\.\|JWT" --include="*.ts" --include="*.js"

# Find sign/verify calls
grep -rn "jwt.sign\|jwt.verify" --include="*.ts" --include="*.js"
```

**Vulnerable patterns:**
```typescript
// ❌ No algorithm specified (allows "none" attack)
const token = jwt.sign(payload, secret);

// ❌ Weak secret
const secret = "mysecret";

// ❌ No expiration
const token = jwt.sign({ userId: user.id }, secret);

// ❌ Client-only logout (token still valid)
app.post('/logout', (req, res) => {
  res.clearCookie('token');
  res.json({ success: true });
});
```

**Secure patterns:**
```typescript
// ✅ Explicit algorithm
const token = jwt.sign(payload, secret, { 
  algorithm: 'HS256',
  expiresIn: '15m'
});

// ✅ Strong secret (256-bit minimum for HS256)
const secret = process.env.JWT_SECRET; // Should be 32+ random bytes

// ✅ With expiration and issuer
const token = jwt.sign(
  { userId: user.id },
  secret,
  {
    algorithm: 'HS256',
    expiresIn: '15m',
    issuer: 'myapp.com'
  }
);

// ✅ Verify with algorithm specification
const decoded = jwt.verify(token, secret, {
  algorithms: ['HS256'], // Prevent algorithm confusion
  issuer: 'myapp.com'
});

// ✅ Server-side logout (token blocklist)
const tokenBlocklist = new Set(); // Use Redis in production

app.post('/logout', authMiddleware, (req, res) => {
  tokenBlocklist.add(req.token);
  res.json({ success: true });
});

// Check blocklist in auth middleware
if (tokenBlocklist.has(token)) {
  throw new UnauthorizedError('Token revoked');
}
```

### Refresh Token Security

```typescript
// ✅ Refresh token rotation
app.post('/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  // Verify refresh token exists and is valid
  const storedToken = await prisma.refreshToken.findUnique({
    where: { token: refreshToken }
  });
  
  if (!storedToken || storedToken.expiresAt < new Date()) {
    return res.status(401).json({ error: 'Invalid refresh token' });
  }
  
  // Rotate: delete old, create new
  await prisma.refreshToken.delete({ where: { id: storedToken.id } });
  
  const newAccessToken = generateAccessToken(storedToken.userId);
  const newRefreshToken = generateRefreshToken();
  
  await prisma.refreshToken.create({
    data: {
      token: newRefreshToken,
      userId: storedToken.userId,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
    }
  });
  
  res.json({ accessToken: newAccessToken, refreshToken: newRefreshToken });
});
```

---

## Authorization Middleware

### Protecting Routes

Check for:
- [ ] Protected routes missing auth middleware
- [ ] Admin routes accessible to regular users
- [ ] Role checks done client-side only

**Vulnerable patterns:**
```typescript
// ❌ No auth middleware
app.get('/api/profile', async (req, res) => {
  const user = await prisma.user.findUnique({ 
    where: { id: req.body.userId } // User-provided ID!
  });
});

// ❌ Auth middleware not applied
router.delete('/users/:id', deleteUser); // Missing authMiddleware

// ❌ Client-only role check
// Frontend: if (user.role === 'admin') showAdminPanel();
// Backend has no check!
```

**Secure patterns:**
```typescript
// ✅ Auth middleware on protected routes
app.get('/api/profile', authMiddleware, async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.user.id } // From verified token
  });
});

// ✅ Role-based middleware
const requireRole = (...roles: string[]) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
};

app.delete('/api/users/:id', 
  authMiddleware, 
  requireRole('ADMIN'),
  deleteUser
);

// ✅ Resource ownership check
app.put('/api/posts/:id', authMiddleware, async (req, res) => {
  const post = await prisma.post.findFirst({
    where: {
      id: req.params.id,
      authorId: req.user.id // Owner only
    }
  });
  
  if (!post) {
    return res.status(404).json({ error: 'Post not found' });
  }
  // Update post...
});
```

---

## Rate Limiting

### Implementing Rate Limits

Check for:
- [ ] No rate limiting on auth endpoints
- [ ] No rate limiting on expensive operations
- [ ] Same limits for authenticated vs anonymous

**Required rate limits:**

| Endpoint Type | Recommended Limit | Risk |
|---------------|-------------------|------|
| Login | 5/min per IP | Brute force |
| Register | 3/min per IP | Spam accounts |
| Password reset | 3/hour per email | Enumeration |
| OTP/2FA | 5/min per user | Brute force |
| API calls | 100/min per user | DoS |
| File upload | 10/min per user | Resource abuse |

**Implementation:**
```typescript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

// ✅ General API rate limit
const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Too many requests, please try again later' }
});

// ✅ Strict auth rate limit
const authLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  skipSuccessfulRequests: false,
  keyGenerator: (req) => req.ip, // By IP for unauthenticated
  message: { error: 'Too many login attempts' }
});

// ✅ Even stricter for password reset
const passwordResetLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 3,
  keyGenerator: (req) => req.body.email || req.ip,
  message: { error: 'Too many password reset requests' }
});

// Apply limiters
app.use('/api/', apiLimiter);
app.post('/api/auth/login', authLimiter, loginHandler);
app.post('/api/auth/register', authLimiter, registerHandler);
app.post('/api/auth/forgot-password', passwordResetLimiter, forgotPasswordHandler);
```

**Search commands:**
```bash
# Check for rate limiting
grep -rn "rate-limit\|rateLimit\|express-rate-limit" --include="*.ts" --include="*.js"
```

---

## Input Validation

### Validating All Input

Check for:
- [ ] Request body used directly without validation
- [ ] Query params used without sanitization
- [ ] File uploads without type/size validation

**Vulnerable patterns:**
```typescript
// ❌ No validation
app.post('/api/users', async (req, res) => {
  const user = await prisma.user.create({
    data: req.body // Could contain anything!
  });
});

// ❌ No type coercion
app.get('/api/items', async (req, res) => {
  const limit = req.query.limit; // String, could be "999999"
  const items = await prisma.item.findMany({ take: limit });
});
```

**Secure patterns with Zod:**
```typescript
import { z } from 'zod';

// ✅ Define schemas
const CreateUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  name: z.string().min(1).max(100)
});

const PaginationSchema = z.object({
  limit: z.coerce.number().int().min(1).max(100).default(20),
  offset: z.coerce.number().int().min(0).default(0)
});

// ✅ Validate in routes
app.post('/api/users', async (req, res) => {
  const result = CreateUserSchema.safeParse(req.body);
  
  if (!result.success) {
    return res.status(400).json({ 
      error: 'Validation failed',
      details: result.error.flatten()
    });
  }
  
  const user = await prisma.user.create({
    data: {
      email: result.data.email,
      password: await hashPassword(result.data.password),
      name: result.data.name
    }
  });
});

// ✅ Validate query params
app.get('/api/items', async (req, res) => {
  const { limit, offset } = PaginationSchema.parse(req.query);
  const items = await prisma.item.findMany({
    take: limit,
    skip: offset
  });
});
```

**File upload validation:**
```typescript
import multer from 'multer';

const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB max
  },
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'image/webp'];
    if (!allowedTypes.includes(file.mimetype)) {
      return cb(new Error('Invalid file type'));
    }
    cb(null, true);
  }
});

app.post('/api/avatar', authMiddleware, upload.single('avatar'), async (req, res) => {
  // File validated by multer
});
```

---

## Error Responses

### Preventing Information Leakage

Check for:
- [ ] Different error messages for valid vs invalid users
- [ ] Database errors exposed to clients
- [ ] Stack traces in production responses

**Account enumeration prevention:**
```typescript
// ❌ Different messages reveal account existence
app.post('/login', async (req, res) => {
  const user = await prisma.user.findUnique({ where: { email } });
  if (!user) {
    return res.status(400).json({ error: 'User not found' }); // Reveals no account!
  }
  if (!await bcrypt.compare(password, user.password)) {
    return res.status(400).json({ error: 'Incorrect password' }); // Reveals account exists!
  }
});

// ✅ Same message for all failures
app.post('/login', async (req, res) => {
  const user = await prisma.user.findUnique({ where: { email } });
  const isValid = user && await bcrypt.compare(password, user.password);
  
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid email or password' });
  }
  // Success...
});

// ✅ Same for password reset
app.post('/forgot-password', async (req, res) => {
  const user = await prisma.user.findUnique({ where: { email } });
  
  if (user) {
    await sendPasswordResetEmail(user);
  }
  
  // Always return same response
  res.json({ message: 'If an account exists, a reset email has been sent' });
});
```

---

## Quick Reference Table

| Check | Severity | Files to Inspect |
|-------|----------|------------------|
| JWT without algorithm | 🔴 Critical | Auth/token files |
| No auth middleware | 🔴 Critical | Route definitions |
| No input validation | 🔴 Critical | All endpoints |
| No rate limiting on auth | 🟠 High | Auth routes |
| Account enumeration | 🟠 High | Login, registration |
| Weak JWT secret | 🟠 High | Environment config |
| Missing role checks | 🟠 High | Admin endpoints |
| No refresh token rotation | 🟡 Medium | Token refresh |
| No file upload limits | 🟡 Medium | Upload endpoints |
