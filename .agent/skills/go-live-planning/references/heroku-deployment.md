# Heroku Production Deployment Checklist

Complete checklist for deploying backend and web app to Heroku production.

## Table of Contents
1. [Pre-Deployment Setup](#pre-deployment-setup)
2. [Environment Configuration](#environment-configuration)
3. [Database Setup](#database-setup)
4. [Application Configuration](#application-configuration)
5. [Deployment Pipeline](#deployment-pipeline)
6. [Monitoring & Logging](#monitoring--logging)
7. [Go-Live Steps](#go-live-steps)

---

## Pre-Deployment Setup

### Heroku Apps Structure

For your stack, create separate Heroku apps:

| App | Purpose | Suggested Name |
|-----|---------|----------------|
| Backend API | Node.js/Express server | `yourapp-api-prod` |
| Web App | Landing page + web portal | `yourapp-web-prod` |

### Heroku Add-ons Required

- [ ] **Heroku Postgres** - Production tier (Standard-0 minimum)
- [ ] **Papertrail** or **Logentries** - Log management
- [ ] **New Relic** or **Scout** - APM monitoring (optional)
- [ ] **Heroku Redis** - Caching/sessions (if needed)
- [ ] **Heroku Scheduler** - Cron jobs (if needed)

```bash
# Create production apps
heroku create yourapp-api-prod
heroku create yourapp-web-prod

# Add PostgreSQL (Standard-0 for production)
heroku addons:create heroku-postgresql:standard-0 -a yourapp-api-prod

# Add logging
heroku addons:create papertrail:choklad -a yourapp-api-prod
heroku addons:create papertrail:choklad -a yourapp-web-prod
```

---

## Environment Configuration

### Backend API Config Vars

```bash
# Set all required environment variables
heroku config:set -a yourapp-api-prod \
  NODE_ENV=production \
  PORT=3000 \
  DATABASE_URL="<auto-set-by-postgres-addon>" \
  
# Firebase
heroku config:set -a yourapp-api-prod \
  FIREBASE_PROJECT_ID=your-project-id \
  FIREBASE_SERVICE_ACCOUNT='<json-escaped-service-account>'

# Stripe
heroku config:set -a yourapp-api-prod \
  STRIPE_SECRET_KEY=sk_live_xxx \
  STRIPE_WEBHOOK_SECRET=whsec_xxx

# RevenueCat
heroku config:set -a yourapp-api-prod \
  REVENUECAT_SECRET_KEY=sk_xxx \
  REVENUECAT_WEBHOOK_SECRET=xxx

# Cloudflare R2
heroku config:set -a yourapp-api-prod \
  CLOUDFLARE_ACCOUNT_ID=xxx \
  R2_ACCESS_KEY_ID=xxx \
  R2_SECRET_ACCESS_KEY=xxx \
  R2_BUCKET_NAME=yourapp-prod

# JWT/Auth
heroku config:set -a yourapp-api-prod \
  JWT_SECRET=<strong-256-bit-secret>
```

### Web App Config Vars

```bash
heroku config:set -a yourapp-web-prod \
  NODE_ENV=production \
  API_URL=https://api.yourdomain.com \
  SOFT_LAUNCH_MODE=true  # For soft launch
```

### Verify All Config Vars Set

```bash
# List all config vars
heroku config -a yourapp-api-prod

# Ensure no placeholder values remain
# Check for: xxx, your-xxx, <placeholder>, TODO
```

---

## Database Setup

### PostgreSQL Production Configuration

- [ ] Use `standard-0` plan minimum (for production SLA)
- [ ] Enable connection pooling for high traffic
- [ ] Set up automated backups schedule

```bash
# Check database info
heroku pg:info -a yourapp-api-prod

# Enable connection pooling
# DATABASE_URL automatically includes pooler if needed

# Verify backups are enabled
heroku pg:backups:schedules -a yourapp-api-prod

# Create manual backup before go-live
heroku pg:backups:capture -a yourapp-api-prod
```

### Prisma Migrations

```bash
# Run migrations on production database
# ⚠️ ALWAYS backup before running migrations!

# Option 1: Deploy migrations
heroku run "npx prisma migrate deploy" -a yourapp-api-prod

# Option 2: If using generate only
heroku run "npx prisma generate" -a yourapp-api-prod
```

### Database Security

- [ ] SSL enforced (`?sslmode=require` in DATABASE_URL)
- [ ] Connection string never logged
- [ ] Database credentials in config vars only

---

## Application Configuration

### Procfile

```procfile
# Procfile for backend
web: node dist/server.js

# If using workers
worker: node dist/worker.js
```

### Production Build

```bash
# Ensure build script produces production code
npm run build

# Verify in package.json
# "build": "tsc" or "build": "next build"
# "start": "node dist/server.js"
```

### Health Check Endpoint

```typescript
// Add health check endpoint for monitoring
app.get('/health', (req, res) => {
  res.status(200).json({ 
    status: 'ok',
    timestamp: new Date().toISOString()
  });
});

// More detailed for internal monitoring
app.get('/health/detailed', async (req, res) => {
  try {
    await prisma.$queryRaw`SELECT 1`;
    res.json({ 
      status: 'ok',
      database: 'connected',
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(503).json({ 
      status: 'error',
      database: 'disconnected'
    });
  }
});
```

---

## Deployment Pipeline

### Heroku Pipeline Setup

```bash
# Create pipeline
heroku pipelines:create yourapp-pipeline

# Add staging app
heroku pipelines:add yourapp-pipeline -a yourapp-api-staging -s staging

# Add production app
heroku pipelines:add yourapp-pipeline -a yourapp-api-prod -s production
```

### GitHub Integration

1. Connect GitHub repo in Heroku Dashboard
2. Enable automatic deploys for staging (optional)
3. Require CI to pass before deploy (recommended)
4. Enable Review Apps for PRs (optional)

### Manual Production Deploy

```bash
# Deploy to production
git push heroku-prod main

# Or promote from staging
heroku pipelines:promote -a yourapp-api-staging
```

---

## Monitoring & Logging

### Essential Monitoring

- [ ] **Error tracking** - Sentry configured
- [ ] **Uptime monitoring** - UptimeRobot or similar
- [ ] **APM** - Response times, throughput
- [ ] **Alerts** - Error rate, response time

### Log Configuration

```bash
# View logs
heroku logs --tail -a yourapp-api-prod

# With Papertrail add-on
heroku addons:open papertrail -a yourapp-api-prod
```

### Set Up Alerts

Configure alerts in your monitoring tool:

| Metric | Threshold | Action |
|--------|-----------|--------|
| Response time | > 2s average | Warning |
| Error rate | > 1% | Critical |
| Memory usage | > 90% | Warning |
| Dyno restarts | > 5/hour | Critical |

---

## Go-Live Steps

### Pre-Go-Live Checklist

- [ ] All config vars set with production values
- [ ] Database backup taken
- [ ] Migrations run successfully
- [ ] Health endpoint responding
- [ ] SSL working on custom domain
- [ ] Error tracking configured
- [ ] Logging working

### Dyno Scaling

```bash
# Scale to production dynos
# Standard-1x minimum for production

# Backend API
heroku ps:scale web=2:standard-1x -a yourapp-api-prod

# Web App
heroku ps:scale web=1:standard-1x -a yourapp-web-prod
```

### DNS Configuration

See `cloudflare-setup.md` for detailed DNS setup.

```bash
# Add custom domain
heroku domains:add api.yourdomain.com -a yourapp-api-prod
heroku domains:add www.yourdomain.com -a yourapp-web-prod

# Get DNS targets
heroku domains -a yourapp-api-prod
# → Configure CNAME to the DNS target shown
```

### Maintenance Mode

```bash
# Enable maintenance mode during critical updates
heroku maintenance:on -a yourapp-api-prod

# Disable when ready
heroku maintenance:off -a yourapp-api-prod
```

---

## Quick Reference

| Item | Status | Notes |
|------|--------|-------|
| Heroku apps created | ⬜ | |
| PostgreSQL provisioned | ⬜ | |
| All config vars set | ⬜ | |
| Migrations run | ⬜ | |
| Health endpoint working | ⬜ | |
| Logging configured | ⬜ | |
| Monitoring configured | ⬜ | |
| Custom domains added | ⬜ | |
| SSL certificates | ⬜ | |
| Dynos scaled | ⬜ | |
| Database backup taken | ⬜ | |
| Deployment tested | ⬜ | |
