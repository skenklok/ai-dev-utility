# Security Review Checklist

Security checks for PR reviews focusing on common vulnerabilities.

## Authentication & Authorization

### Authentication
- [ ] JWT secret strength and algorithm validation
- [ ] Token expiration properly enforced
- [ ] Session invalidation on logout
- [ ] Account enumeration prevented (same response for valid/invalid)
- [ ] Password reset flow secure (token expiration, single use)
- [ ] OAuth state parameter validated

### Authorization
- [ ] IDOR (Insecure Direct Object Reference) checks
  - User context included in all data queries
  - Can't access other users' data by changing IDs
- [ ] Function-level authorization
  - Admin routes protected
  - Role checks on sensitive operations
- [ ] Mass assignment prevention
  - Only allowed fields accepted in updates

## Input Validation & Injection

### SQL/NoSQL Injection
```bash
# Search for raw queries
grep -r "executeRaw\|queryRaw\|$where\|eval("
```
- [ ] All user input parameterized
- [ ] No string concatenation in queries
- [ ] ORM used for database operations

### XSS (Cross-Site Scripting)
```bash
# Search for dangerous patterns
grep -r "dangerouslySetInnerHTML\|innerHTML\|v-html"
```
- [ ] User input escaped before rendering
- [ ] Content Security Policy headers set
- [ ] HttpOnly cookies for sensitive tokens

### Command Injection
```bash
# Search for shell execution
grep -r "exec(\|spawn(\|child_process"
```
- [ ] No user input in shell commands
- [ ] If necessary, input strictly validated

## Data Exposure

### Sensitive Data in Code
```bash
# Search for secrets
grep -ri "password\|secret\|api_key\|token" --include="*.ts" --include="*.js"
```
- [ ] No hardcoded credentials
- [ ] Secrets in environment variables only
- [ ] .env files in .gitignore

### Sensitive Data in Logs
```bash
# Check logging
grep -r "console.log\|logger\." | grep -i "password\|token\|secret"
```
- [ ] No passwords/tokens logged
- [ ] No PII in logs
- [ ] Log levels appropriate for production

### API Response Data
- [ ] No sensitive fields in API responses
- [ ] User passwords never returned
- [ ] Internal IDs hidden if needed

## Rate Limiting

Check rate limiting on:
- [ ] Authentication endpoints (login, register, password reset)
- [ ] Email/SMS sending endpoints
- [ ] File upload endpoints
- [ ] Payment operations
- [ ] Resource-intensive operations
- [ ] Public APIs

## SSRF/CSRF

### SSRF (Server-Side Request Forgery)
- [ ] User-controlled URLs validated
- [ ] Internal network access blocked
- [ ] Allowlist for external services

### CSRF (Cross-Site Request Forgery)
- [ ] CSRF tokens on state-changing requests
- [ ] SameSite cookie attribute set
- [ ] Origin/Referer validation

## Quick Reference

| Vulnerability | Severity | Common Location |
|--------------|----------|-----------------|
| SQL Injection | 🔴 Critical | Database queries |
| Auth Bypass | 🔴 Critical | Auth middleware |
| IDOR | 🔴 Critical | Data access routes |
| XSS | 🟠 High | User content display |
| Hardcoded Secrets | 🟠 High | Config files |
| Missing Rate Limit | 🟡 Medium | Auth endpoints |
| Sensitive Logging | 🟡 Medium | Logger calls |
