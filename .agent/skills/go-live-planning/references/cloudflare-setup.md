# Cloudflare Setup Checklist

Complete checklist for Cloudflare DNS, SSL, R2 storage, and security configuration.

## Table of Contents
1. [Domain Setup](#domain-setup)
2. [DNS Configuration](#dns-configuration)
3. [SSL/TLS Settings](#ssltls-settings)
4. [R2 Storage](#r2-storage)
5. [Security Settings](#security-settings)
6. [Performance Settings](#performance-settings)

---

## Domain Setup

### If Domain Purchased from Cloudflare

Your domain is already using Cloudflare nameservers. Skip to DNS Configuration.

### If Domain from Another Registrar

1. Add domain to Cloudflare account
2. Cloudflare will scan existing DNS records
3. Update nameservers at your registrar to Cloudflare's:
   - `ns1.cloudflare.com` (example - use your assigned ones)
   - `ns2.cloudflare.com`

**Verification:**
```bash
# Check nameserver propagation (may take 24-48 hours)
dig NS yourdomain.com
```

---

## DNS Configuration

### Required DNS Records

For your Heroku apps, create these records:

| Type | Name | Content | Proxy |
|------|------|---------|-------|
| CNAME | `api` | `yourapp-api-prod.herokuapp.com` | ✅ Proxied |
| CNAME | `www` | `yourapp-web-prod.herokuapp.com` | ✅ Proxied |
| CNAME | `@` (root) | `yourapp-web-prod.herokuapp.com` | ✅ Proxied |

**Note:** Cloudflare uses CNAME flattening for root domain, so CNAME at `@` works.

### Creating Records

1. Go to Cloudflare Dashboard → Your Domain → DNS
2. Click "Add record"
3. For each record:
   - Type: CNAME
   - Name: subdomain (e.g., `api`)
   - Target: Heroku DNS target
   - Proxy status: Proxied (orange cloud)

### Heroku Domain Setup

```bash
# Add custom domains to Heroku apps
heroku domains:add api.yourdomain.com -a yourapp-api-prod
heroku domains:add www.yourdomain.com -a yourapp-web-prod
heroku domains:add yourdomain.com -a yourapp-web-prod

# Get DNS targets (use these in Cloudflare)
heroku domains -a yourapp-api-prod
# Example output:
# api.yourdomain.com    →  hollow-wilderbeest-xxx.herokudns.com
```

### Wait for Propagation

```bash
# Verify DNS is working
dig api.yourdomain.com
dig www.yourdomain.com

# Should resolve to Cloudflare IPs (not Heroku directly)
```

---

## SSL/TLS Settings

### Recommended SSL Mode

**⚠️ Important: Use "Full (strict)" for production**

1. Go to Cloudflare Dashboard → SSL/TLS → Overview
2. Set encryption mode to **Full (strict)**

| Mode | Description | Use Case |
|------|-------------|----------|
| Off | No encryption | ❌ Never use |
| Flexible | Cloudflare to user only | ❌ Not secure |
| Full | End-to-end, any cert | ⚠️ Minimum |
| Full (strict) | End-to-end, valid cert | ✅ Production |

### Edge Certificates

Cloudflare provides free SSL certificates automatically.

- [ ] Universal SSL enabled (default)
- [ ] Covers `yourdomain.com` and `*.yourdomain.com`

### Additional SSL Settings

Go to SSL/TLS → Edge Certificates:

- [ ] **Always Use HTTPS**: ON
- [ ] **HTTP Strict Transport Security (HSTS)**: Enable
  - Max Age: 12 months
  - Include subdomains: Yes
  - Preload: Yes (after testing)
- [ ] **Minimum TLS Version**: 1.2
- [ ] **Automatic HTTPS Rewrites**: ON

---

## R2 Storage

### Create R2 Bucket

1. Go to Cloudflare Dashboard → R2
2. Click "Create bucket"
3. Name: `yourapp-prod` (or similar)
4. Location: Auto or specific region

### Create API Token for R2

1. Go to R2 → Manage R2 API Tokens
2. Create token with permissions:
   - Object Read & Write (for your bucket)

```bash
# Store these in Heroku config vars
heroku config:set -a yourapp-api-prod \
  CLOUDFLARE_ACCOUNT_ID=your-account-id \
  R2_ACCESS_KEY_ID=your-access-key \
  R2_SECRET_ACCESS_KEY=your-secret-key \
  R2_BUCKET_NAME=yourapp-prod
```

### R2 Configuration in Code

```typescript
import { S3Client } from '@aws-sdk/client-s3';

const r2Client = new S3Client({
  region: 'auto',
  endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!
  }
});
```

### Public Access (If Needed)

For public assets (like images), you can:

1. **Custom domain** (recommended):
   - R2 → Bucket → Settings → Public access
   - Connect custom domain (e.g., `assets.yourdomain.com`)

2. **R2.dev subdomain**:
   - Enable public access via r2.dev

---

## Security Settings

### Firewall Rules

Go to Security → WAF:

- [ ] **Managed Rulesets**: Enable OWASP Core Ruleset
- [ ] **Rate Limiting**: Configure for API endpoints

### Rate Limiting Example

For API protection:

```
# Rule: Protect auth endpoints
# Expression: (http.request.uri.path contains "/api/auth")
# Action: Block
# Rate: 20 requests per minute
```

### Security Headers

Go to Rules → Transform Rules → Modify Response Header:

Add these headers for all responses:

| Header | Value |
|--------|-------|
| X-Content-Type-Options | nosniff |
| X-Frame-Options | DENY |
| Referrer-Policy | strict-origin-when-cross-origin |

**Note:** Some headers (like CSP) are better set in your application.

### Bot Protection

Go to Security → Bots:

- [ ] **Bot Fight Mode**: ON (for free plan)
- [ ] **Super Bot Fight Mode**: Configure (for Pro+)

### DDoS Protection

Automatically enabled, but verify:

- [ ] Go to Security → DDoS
- [ ] Review sensitivity levels (default is fine for most apps)

---

## Performance Settings

### Caching

Go to Caching → Configuration:

- [ ] **Caching Level**: Standard
- [ ] **Browser Cache TTL**: Respect Existing Headers

### Cache Rules for Static Assets

Create cache rule for static assets:

```
# Expression: (http.request.uri.path contains "/static/")
# Cache eligibility: Eligible for cache
# Edge TTL: 1 month
```

### Disable Caching for API

```
# Expression: (http.request.uri.path contains "/api/")
# Cache eligibility: Bypass cache
```

### Speed Optimizations

Go to Speed → Optimization:

- [ ] **Auto Minify**: Enable for JavaScript, CSS, HTML (web app)
- [ ] **Brotli**: ON
- [ ] **Early Hints**: ON
- [ ] **Rocket Loader**: Test before enabling

### Image Optimization (Pro+)

If on Pro plan or higher:

- [ ] **Polish**: Lossless or Lossy
- [ ] **Mirage**: ON for mobile optimization
- [ ] **WebP**: ON

---

## DNSSEC

### Enable DNSSEC

1. Go to DNS → Settings
2. Enable DNSSEC
3. Add DS record at your registrar (if domain not from Cloudflare)

**Note:** If domain is from Cloudflare, DNSSEC is handled automatically.

---

## Quick Reference

| Item | Status | Notes |
|------|--------|-------|
| Domain added to Cloudflare | ⬜ | |
| Nameservers updated | ⬜ | |
| DNS records created | ⬜ | |
| Heroku domains configured | ⬜ | |
| SSL mode set to Full (strict) | ⬜ | |
| Always Use HTTPS enabled | ⬜ | |
| HSTS enabled | ⬜ | |
| R2 bucket created | ⬜ | |
| R2 API token created | ⬜ | |
| WAF rules configured | ⬜ | |
| Rate limiting set up | ⬜ | |
| Caching configured | ⬜ | |
| DNSSEC enabled | ⬜ | |
