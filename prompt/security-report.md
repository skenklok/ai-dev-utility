# Security Audit Prompt

You are my senior security engineer and penetration testing expert. Perform a comprehensive security audit of this codebase.

## TECH STACK CONTEXT
This is a React Native mobile application with:
- **Backend**: Node.js/Express deployed on Heroku
- **Database**: PostgreSQL via Prisma ORM
- **Payments**: Stripe integration + RevenueCat for subscription management
- **In-App Purchases**: Apple App Store and Google Play Store subscriptions
- **Authentication**: Firebase + JWT-based auth (specify if using OAuth/Social login)

Think step-by-step through each security category. For each finding, explain WHY it's a vulnerability and HOW an attacker could exploit it.

---

## AUDIT SCOPE

### 1. INPUT SANITIZATION & INJECTION ATTACKS

**Scan for:**
- All user input points (forms, URL params, API bodies, headers, file uploads, deep links)
- Unsanitized inputs passed to dangerous operations

**Check:**
- [ ] SQL/Prisma injection: Flag `$executeRaw`, `$queryRaw`, `$executeRawUnsafe`, `$queryRawUnsafe` without parameterization
- [ ] NoSQL injection patterns
- [ ] XSS: `dangerouslySetInnerHTML`, unescaped template literals in React Native WebViews
- [ ] Command injection: `exec()`, `spawn()`, `child_process` with user input
- [ ] Path traversal: `fs` operations with user-controlled paths
- [ ] SSRF: User-controlled URLs in fetch/axios calls
- [ ] Deep link injection: Unvalidated deep link parameters in React Native

---

### 2. AUTHENTICATION & SESSION SECURITY

**JWT Security:**
- [ ] Algorithm explicitly specified (vulnerable to "none" algorithm attacks)
- [ ] Secret strength (min 256-bit for HS256)
- [ ] Token expiration (`exp` claim) enforced
- [ ] Refresh token rotation implemented
- [ ] Tokens invalidated server-side on logout (not just client-side)

**Login/Registration:**
- [ ] Account enumeration prevention (same response for valid/invalid users)
- [ ] Brute force protection on login endpoints
- [ ] Password complexity requirements enforced
- [ ] Secure password reset flow (time-limited, single-use tokens)
- [ ] MFA implementation (if applicable)

**OAuth/Social Auth:**
- [ ] State parameter used to prevent CSRF
- [ ] Token validation with provider (not just trusting client tokens)
- [ ] Proper scope limitations

---

### 3. AUTHORIZATION & ACCESS CONTROL

**Check for:**
- [ ] IDOR (Insecure Direct Object Reference): Prisma queries missing user context filters
- [ ] Broken Function Level Authorization: Admin endpoints accessible to regular users
- [ ] Missing authorization middleware on protected routes
- [ ] Horizontal privilege escalation (accessing other users' data)
- [ ] Vertical privilege escalation (regular user → admin)

**Prisma/Database specific:**
- [ ] Multi-tenant data isolation in queries
- [ ] Soft-deleted records accessible via direct query
- [ ] Mass assignment vulnerabilities in `.create()` / `.update()` with unsanitized objects

---

### 4. RATE LIMITING & ABUSE PREVENTION

**List all endpoints and check for rate limiting on:**
- [ ] Authentication endpoints (login, register, password reset, OTP verification)
- [ ] Email/SMS sending endpoints
- [ ] File upload endpoints
- [ ] Payment/subscription operations
- [ ] AI/LLM API calls (if any)
- [ ] Resource-intensive operations (reports, exports, searches)
- [ ] Public APIs exposed to mobile apps

**Flag:**
- Endpoints without rate limiting
- Rate limits that are too permissive
- Missing IP-based and user-based limiting

---

### 5. MOBILE APP SECURITY (React Native - OWASP MASVS)

**Credential & Secret Storage:**
- [ ] **CRITICAL**: Secrets in JavaScript bundle (API keys, tokens in source code)
- [ ] Sensitive data in `AsyncStorage` (should use `react-native-keychain` or `expo-secure-store`)
- [ ] Hardcoded credentials in `.env` files bundled with app
- [ ] API keys exposed client-side that should be server-side only

**Code Security:**
- [ ] Source maps enabled in production builds
- [ ] JavaScript obfuscation missing
- [ ] Debug code paths reachable in production
- [ ] ProGuard/R8 not enabled for Android release builds

**Network Security:**
- [ ] SSL/TLS certificate pinning not implemented
- [ ] HTTP endpoints used instead of HTTPS
- [ ] Sensitive data transmitted over insecure channels

**Deep Links:**
- [ ] Deep link parameters not validated before use
- [ ] Sensitive actions triggered via deep links without re-authentication

---

### 6. PAYMENT & SUBSCRIPTION SECURITY (RevenueCat + Stripe + IAP)

**In-App Purchase Security (CRITICAL):**
- [ ] **CRITICAL**: Client-only receipt validation (receipts MUST be validated server-side)
- [ ] RevenueCat webhook signature verification missing
- [ ] Entitlement checks done only on client (must verify server-side)
- [ ] Premium features accessible without valid entitlements

**RevenueCat Integration:**
- [ ] Public API key used where secret key is required
- [ ] Server-to-server API calls missing proper authentication
- [ ] Webhook endpoint without signature verification
- [ ] User ID spoofing possible (verify user identity server-side)

**Stripe Integration:**
- [ ] Secret API key exposed to client
- [ ] Payment intents created client-side (should be server-side only)
- [ ] Webhook signature verification missing (`stripe.webhooks.constructEvent`)
- [ ] Raw card data handled by app (should use Stripe SDK only)
- [ ] `success_url` used for fulfillment without server verification

**Subscription Edge Cases:**
- [ ] Race conditions during purchase flows
- [ ] Refund/cancellation not properly revoking access
- [ ] Subscription status not refreshed before granting access
- [ ] Trial abuse not prevented

---

### 7. DATABASE SECURITY (Prisma + PostgreSQL)

**Query Security:**
- [ ] Raw queries without parameterization
- [ ] User input directly concatenated in queries
- [ ] IDOR via missing tenant/user filters in queries

**Connection Security:**
- [ ] DATABASE_URL logged or exposed in errors
- [ ] SSL not enforced (`?sslmode=require` missing)
- [ ] Connection string committed to version control

**Schema Security:**
- [ ] Sensitive data stored unencrypted (passwords as plaintext)
- [ ] PII not properly segregated
- [ ] Audit trails missing for sensitive operations

**Access Control:**
- [ ] Database user has excessive privileges (should use least privilege)
- [ ] No row-level security for multi-tenant data

---

### 8. DEPLOYMENT & INFRASTRUCTURE (Heroku)

**Configuration:**
- [ ] Secrets in code instead of Heroku Config Vars
- [ ] `NODE_ENV` not set to 'production'
- [ ] Debug endpoints accessible in production
- [ ] Review apps with access to production data

**Network:**
- [ ] HTTPS not enforced (missing redirect from HTTP)
- [ ] Missing `SECURE_PROXY_SSL_HEADER` or `trust proxy` configuration
- [ ] CORS too permissive (allows `*`)

**Security Headers:**
- [ ] Content-Security-Policy (CSP) missing or too permissive
- [ ] Strict-Transport-Security (HSTS) missing
- [ ] X-Frame-Options missing
- [ ] X-Content-Type-Options missing
- [ ] Referrer-Policy not set

**Cookies (if applicable):**
- [ ] `Secure` flag missing
- [ ] `HttpOnly` flag missing
- [ ] `SameSite` not set to 'Strict' or 'Lax'

---

### 9. SENSITIVE DATA & LOGGING

**Search for:**
- All logging statements: `console.log`, `console.error`, `logger.*`, Sentry calls

**Flag logging of:**
- [ ] Passwords or password hashes
- [ ] API keys, tokens, secrets
- [ ] Full credit card numbers (only last 4 allowed)
- [ ] Session tokens, JWTs
- [ ] PII (email, phone, SSN, addresses)
- [ ] Request/response bodies containing sensitive data

**Check:**
- [ ] Different log levels for production vs development
- [ ] Sensitive data filtered before sending to error tracking (Sentry, etc.)

---

### 10. DEPENDENCY VULNERABILITIES

**Run:**
```bash
npm audit
```

**Check:**
- [ ] Packages with known CVEs (Critical and High severity)
- [ ] Outdated packages with security patches available
- [ ] Abandoned/unmaintained packages (no updates in 2+ years)
- [ ] Suspicious or typosquatted package names

---

### 11. ERROR HANDLING & INFORMATION DISCLOSURE

**Check error responses for leakage of:**
- [ ] Database connection strings
- [ ] Stack traces with file paths
- [ ] Internal server details (versions, IPs)
- [ ] SQL/Prisma query structures
- [ ] API keys or tokens in error messages

**Verify:**
- [ ] Generic error messages returned to clients
- [ ] Detailed errors logged server-side only
- [ ] Proper try/catch wrapping around async operations

---

### 12. SECRETS & CONFIGURATION

**Search entire codebase for:**
- [ ] Hardcoded API keys, passwords, tokens
- [ ] Private keys or certificates committed
- [ ] `.env` files committed to git
- [ ] Secrets in comments or TODOs
- [ ] Base64-encoded secrets (easy to decode)

**Check:**
- [ ] `.gitignore` includes all sensitive files
- [ ] No secrets in CI/CD pipeline logs
- [ ] Different secrets for development/staging/production

---

## OUTPUT FORMAT

### A. Executive Summary
| Severity | Count | Examples |
|----------|-------|----------|
| 🔴 Critical | X | [Brief list] |
| 🟠 High | X | [Brief list] |
| 🟡 Medium | X | [Brief list] |
| 🟢 Low | X | [Brief list] |

**Overall Risk Assessment:** [CRITICAL/HIGH/MEDIUM/LOW]
**Recommendation:** [BLOCK DEPLOY / FIX BEFORE DEPLOY / FIX IN NEXT SPRINT / ACCEPTABLE RISK]

---

### B. Critical Findings (Must fix before deploy)
For each finding:

**[FINDING-001] [Title]**
- **File:Line**: `path/to/file.ts:123`
- **Type**: [SQL Injection / XSS / Auth Bypass / etc.]
- **Risk**: Why this matters and how an attacker could exploit it
- **Proof of Concept**: (if applicable) Steps to reproduce
- **Fix**: Specific code change with example

```typescript
// ❌ Vulnerable code
const user = await prisma.$queryRaw`SELECT * FROM users WHERE id = ${userId}`;

// ✅ Fixed code
const user = await prisma.user.findUnique({ where: { id: userId } });
```

---

### C. High Priority Findings (Fix within current sprint)
[Same format as Critical]

---

### D. Medium/Low Findings (Fix when possible)
[Same format, can be more condensed]

---

### E. Rate Limiting Gap Analysis

| Endpoint | Method | Current Limit | Recommended Limit | Risk |
|----------|--------|---------------|-------------------|------|
| /api/auth/login | POST | None | 5/min per IP | Brute force |
| ... | ... | ... | ... | ... |

---

### F. Dependency Vulnerability Report

| Package | Current | Patched | Severity | CVE | Impact |
|---------|---------|---------|----------|-----|--------|
| lodash | 4.17.19 | 4.17.21 | High | CVE-XXX | Prototype pollution |
| ... | ... | ... | ... | ... | ... |

---

### G. Payment Security Checklist

| Check | Status | Notes |
|-------|--------|-------|
| Server-side receipt validation | ✅/❌ | |
| RevenueCat webhook signatures | ✅/❌ | |
| Stripe webhook signatures | ✅/❌ | |
| Secret key exposure | ✅/❌ | |
| Entitlement server verification | ✅/❌ | |

---

### H. Mobile Security Checklist (OWASP MASVS)

| Category | Check | Status |
|----------|-------|--------|
| Storage | Secure credential storage | ✅/❌ |
| Storage | No secrets in bundle | ✅/❌ |
| Crypto | Proper encryption used | ✅/❌ |
| Network | Certificate pinning | ✅/❌ |
| Code | Obfuscation enabled | ✅/❌ |

---

### I. Remediation Priority Matrix

| Priority | Finding IDs | Effort | Timeline |
|----------|-------------|--------|----------|
| P0 - Block Deploy | FINDING-001, FINDING-002 | 2-4 hours | Immediate |
| P1 - This Sprint | FINDING-003-005 | 1-2 days | This week |
| P2 - Next Sprint | FINDING-006-010 | 3-5 days | Next sprint |
| P3 - Backlog | FINDING-011+ | Variable | When capacity |

---

## REFERENCES

- OWASP Mobile Top 10 2024: https://owasp.org/www-project-mobile-top-10/
- OWASP MASVS: https://mas.owasp.org/MASVS/
- RevenueCat Security: https://www.revenuecat.com/docs/security
- Stripe Security Guide: https://stripe.com/docs/security
- Prisma Security Best Practices: https://www.prisma.io/docs/guides/security