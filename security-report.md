You are my senior security engineer. Perform a comprehensive security audit of this codebase.

## AUDIT SCOPE
Scan all code in the repository for the following security vulnerabilities:

### 1. INPUT SANITIZATION
- Identify ALL user input points (forms, URL params, API bodies, headers, file uploads)
- Check if inputs are sanitized before:
  - Database queries (SQL injection)
  - HTML rendering (XSS)
  - Command execution (command injection)
  - File path operations (path traversal)
- Flag any raw user input passed directly to: Prisma/SQL, eval(), dangerouslySetInnerHTML, exec(), fs operations

### 2. RATE LIMITING
- List all public endpoints and user-triggered actions
- Check for rate limiting on:
  - Authentication endpoints (login, register, password reset)
  - Email/SMS sending
  - File uploads
  - AI/LLM API calls
  - Payment operations
  - Any resource-intensive endpoints
- Flag endpoints without rate limiting protection

### 3. SENSITIVE DATA LOGGING
- Search for console.log, console.error, logger.*, print statements
- Flag any logging of:
  - Passwords or password hashes
  - API keys, tokens, secrets
  - Email addresses, phone numbers
  - Credit card numbers
  - Session tokens, JWTs
  - Personal identifiable information (PII)
- Check log configuration for production vs development

### 4. DEPENDENCY VULNERABILITIES
- Run `npm audit` analysis
- Identify packages with known CVEs
- Check for outdated dependencies with security patches available
- Flag any abandoned or unmaintained packages

### 5. ERROR HANDLING
- Check error responses returned to clients for information leakage:
  - Database connection strings
  - Stack traces with file paths
  - Internal server details
  - Query structures
- Verify error messages are generic for users but detailed in server logs
- Check try/catch blocks for proper error handling

### 6. ADDITIONAL CHECKS
- Authentication bypass vectors
- Authorization checks on all protected routes
- CORS configuration
- Security headers (CSP, HSTS, X-Frame-Options)
- Secure cookie flags
- Environment variable exposure
- Hardcoded secrets in code

## OUTPUT FORMAT
Produce a security report with:

A. **Executive Summary** - Critical/High/Medium/Low counts

B. **Critical Findings** (must fix before deploy)
   - Issue | File:Line | Risk | Fix

C. **High Priority Findings** (fix within sprint)
   - Issue | File:Line | Risk | Fix

D. **Medium/Low Findings** (fix when possible)
   - Issue | File:Line | Risk | Fix

E. **Rate Limiting Gaps** - Table of endpoints needing limits

F. **Dependency Report** - Vulnerable packages with severity

G. **Remediation Summary** - Prioritized fix list with effort estimates

For each finding, provide:
1. Code location (file:line)
2. Vulnerability type
3. Risk level (Critical/High/Medium/Low)
4. Proof of concept (if applicable)
5. Specific fix with code example