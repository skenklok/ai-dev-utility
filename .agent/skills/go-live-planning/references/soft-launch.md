# Soft Launch Strategy

Strategies for launching with limited access - landing page live, app/portal behind credentials.

## Soft Launch Options

### Option 1: Route-Based Protection (Recommended)

```typescript
// middleware/softLaunch.ts
const PUBLIC_PATHS = ['/', '/landing', '/about', '/privacy', '/terms', '/coming-soon'];

export const softLaunchMiddleware = (req, res, next) => {
  if (process.env.SOFT_LAUNCH_MODE !== 'true') {
    return next(); // Full launch - allow all
  }
  
  if (PUBLIC_PATHS.some(path => req.path === path || req.path.startsWith(path))) {
    return next(); // Public routes
  }
  
  // Protected routes - redirect to coming soon
  return res.redirect('/coming-soon');
};
```

**Heroku config:**
```bash
heroku config:set SOFT_LAUNCH_MODE=true -a yourapp-web-prod

# When ready for full launch:
heroku config:set SOFT_LAUNCH_MODE=false -a yourapp-web-prod
```

---

### Option 2: Basic Auth for Portal

Protect entire `/app` section with basic auth:

```typescript
import basicAuth from 'express-basic-auth';

const portalAuth = basicAuth({
  users: { 'beta': process.env.BETA_PASSWORD },
  challenge: true
});

// Only apply to portal routes
app.use('/app', portalAuth);
```

**Heroku config:**
```bash
heroku config:set BETA_PASSWORD=your-secure-password -a yourapp-web-prod
```

---

### Option 3: Invite-Only Access

Allow only whitelisted users:

```typescript
const allowedEmails = process.env.ALLOWED_EMAILS?.split(',') || [];

const inviteOnlyMiddleware = async (req, res, next) => {
  if (process.env.INVITE_ONLY !== 'true') return next();
  
  if (!req.user || !allowedEmails.includes(req.user.email)) {
    return res.redirect('/waitlist');
  }
  next();
};
```

---

## Mobile App Soft Launch

### iOS: TestFlight

- Up to 10,000 external testers
- No App Store review for updates
- 90-day build expiration

### Android: Closed Testing

- Invite-only via email/Google Group
- No Play Store visibility
- Can transition to production when ready

---

## Staged Rollout Timeline

| Phase | Duration | Scope |
|-------|----------|-------|
| 1. Internal | 1 week | Team only |
| 2. Beta | 2 weeks | Invited users |
| 3. Soft Launch | 1-2 weeks | Landing page public |
| 4. Full Launch | - | All features public |

---

## Switching to Full Launch

```bash
# Web App
heroku config:set SOFT_LAUNCH_MODE=false -a yourapp-web-prod

# Mobile
# iOS: Submit to App Store from TestFlight
# Android: Promote from closed testing to production
```

---

## Quick Reference

| Item | Status |
|------|--------|
| Soft launch middleware implemented | ⬜ |
| Coming soon page created | ⬜ |
| Environment variables set | ⬜ |
| TestFlight configured (iOS) | ⬜ |
| Closed testing set up (Android) | ⬜ |
| Launch date communicated | ⬜ |
