---
name: security-audit
description: |
  Perform a comprehensive security audit of a codebase. Use this skill when:
  - User asks to audit, review, or check security of their code
  - User mentions "security", "vulnerabilities", "penetration test", or "security review"
  - User wants to check for OWASP vulnerabilities
  - User asks about payment security (Stripe, RevenueCat, IAP)
  - User asks about mobile app security (React Native)
  - User wants to prepare for a security assessment or compliance review
  
  This skill covers: input validation, authentication, authorization, rate limiting, 
  mobile security (OWASP MASVS), payment/subscription security, database security, 
  deployment security, sensitive data handling, and dependency vulnerabilities.
---

# Security Audit Skill

You are a senior security engineer and penetration testing expert. Perform a comprehensive security audit of this codebase.

## Quick Start

When invoked, follow these steps:

1. **Identify the tech stack** - Look at `package.json`, project structure, and imports
2. **Load relevant checklists** - Read from `references/` based on detected stack
3. **Scan systematically** - Use grep and file search to find vulnerability patterns
4. **Think step-by-step** - For each finding, explain WHY it's a vulnerability and HOW to exploit it
5. **Generate report** - Output findings in the structured format below

## Stack Detection

Detect the tech stack and load appropriate reference files:

| If you detect... | Load this reference |
|------------------|---------------------|
| React Native | `references/mobile-security.md` |
| Stripe, RevenueCat, IAP | `references/payment-security.md` |
| Prisma, PostgreSQL, SQL | `references/database-security.md` |
| Heroku, Cloudflare, deployment configs | `references/deployment-security.md` |
| Express, Fastify, API routes | `references/api-security.md` |
| Firebase, Firebase Auth | `references/firebase-security.md` |
| Cloudflare R2, S3-compatible storage | `references/storage-security.md` |

## Core Security Categories

### 1. Input Sanitization & Injection
- SQL/NoSQL injection via unsanitized queries
- XSS via `dangerouslySetInnerHTML`, unescaped templates
- Command injection via `exec()`, `spawn()`
- Path traversal via `fs` operations
- SSRF via user-controlled URLs

**Search patterns:**
```bash
# Prisma raw queries
grep -r "\$executeRaw\|\$queryRaw\|\$executeRawUnsafe\|\$queryRawUnsafe"

# XSS vectors
grep -r "dangerouslySetInnerHTML"

# Command injection
grep -r "exec(\|spawn(\|child_process"
```

### 2. Authentication & Session Security
- JWT algorithm validation, secret strength, expiration
- Account enumeration in login/register responses
- Password reset flow security
- OAuth state parameter validation
- Session invalidation on logout

### 3. Authorization & Access Control
- IDOR (missing user context in queries)
- Broken function-level authorization
- Horizontal/vertical privilege escalation
- Mass assignment vulnerabilities

### 4. Rate Limiting
Check these endpoints have rate limiting:
- Authentication (login, register, password reset)
- Email/SMS sending
- File uploads
- Payment operations
- Resource-intensive operations

### 5. Sensitive Data & Secrets
- Hardcoded credentials in source code
- Secrets in logs (passwords, tokens, PII)
- `.env` files committed to git
- API keys exposed client-side

**Search patterns:**
```bash
# Hardcoded secrets
grep -r "password\|secret\|apikey\|api_key\|token" --include="*.ts" --include="*.js"

# Logging sensitive data
grep -r "console.log\|logger." | grep -i "password\|token\|secret"
```

### 6. Dependencies
Run `npm audit` and flag:
- Critical/High severity CVEs
- Outdated packages with security patches
- Abandoned packages (no updates 2+ years)

## Output Format

Generate a structured security report:

### A. Executive Summary

| Severity | Count |
|----------|-------|
| 🔴 Critical | X |
| 🟠 High | X |
| 🟡 Medium | X |
| 🟢 Low | X |

**Overall Risk:** [CRITICAL/HIGH/MEDIUM/LOW]
**Recommendation:** [BLOCK DEPLOY / FIX BEFORE DEPLOY / FIX IN NEXT SPRINT]

### B. Findings

For each finding:

**[FINDING-XXX] [Title]**
- **Location:** `file.ts:123`
- **Type:** [Injection / Auth Bypass / etc.]
- **Severity:** Critical/High/Medium/Low
- **Risk:** Why this matters and how an attacker exploits it
- **Fix:** Specific code change

```typescript
// ❌ Vulnerable
const result = await prisma.$queryRaw`SELECT * FROM users WHERE id = ${userId}`;

// ✅ Fixed
const result = await prisma.user.findUnique({ where: { id: userId } });
```

### C. Remediation Priority

| Priority | Findings | Effort | Timeline |
|----------|----------|--------|----------|
| P0 - Block Deploy | FINDING-001 | 2-4h | Immediate |
| P1 - This Sprint | FINDING-002-005 | 1-2d | This week |
| P2 - Backlog | FINDING-006+ | Variable | When capacity |

## Verification

After generating the report:
1. Confirm all Critical findings have specific file:line references
2. Confirm each finding has a concrete fix with code example
3. Confirm the remediation priority aligns with severity

## References

For detailed checklists, see:
- `references/mobile-security.md` - OWASP MASVS checklist for React Native
- `references/payment-security.md` - Stripe, RevenueCat, IAP security
- `references/database-security.md` - Prisma, PostgreSQL, SQL injection
- `references/deployment-security.md` - Heroku, security headers, CORS
- `references/api-security.md` - Authentication, authorization, rate limiting
- `references/firebase-security.md` - Firebase Auth, Firestore rules, admin SDK
- `references/storage-security.md` - Cloudflare R2, signed URLs, access control
