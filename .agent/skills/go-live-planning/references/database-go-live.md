# Database Go-Live Checklist

PostgreSQL + Prisma production database setup.

## Pre-Production

### Database Tier

| Plan | Rows | Connections | Use |
|------|------|-------------|-----|
| Essential-0 | 10K | 20 | Dev only |
| Standard-0 | Unlimited | 120 | Production ✅ |

```bash
heroku pg:info -a yourapp-api-prod
heroku addons:upgrade heroku-postgresql:standard-0 -a yourapp-api-prod
```

---

## Migration Strategy

```bash
# 1. Backup first
heroku pg:backups:capture -a yourapp-api-prod

# 2. Maintenance mode (if destructive)
heroku maintenance:on -a yourapp-api-prod

# 3. Run migrations
heroku run "npx prisma migrate deploy" -a yourapp-api-prod

# 4. Verify
heroku run "npx prisma migrate status" -a yourapp-api-prod

# 5. Disable maintenance
heroku maintenance:off -a yourapp-api-prod
```

### Zero-Downtime Pattern

1. Add optional columns first
2. Deploy and backfill
3. Make required in separate migration

---

## Backup & Recovery

```bash
# Schedule daily backup
heroku pg:backups:schedule DATABASE_URL --at '03:00 UTC' -a yourapp-api-prod

# Manual backup
heroku pg:backups:capture -a yourapp-api-prod

# Download locally
heroku pg:backups:download -a yourapp-api-prod

# Restore if needed
heroku pg:backups:restore b001 DATABASE_URL -a yourapp-api-prod
```

---

## Performance

### Index Checklist
- [ ] Foreign keys indexed
- [ ] Frequently queried columns indexed
- [ ] ORDER BY columns indexed

```bash
# Analyze queries
heroku pg:diagnose -a yourapp-api-prod
heroku pg:outliers -a yourapp-api-prod
```

---

## Quick Reference

| Item | Status |
|------|--------|
| Database plan selected | ⬜ |
| Migrations tested on staging | ⬜ |
| Indexes created | ⬜ |
| Backup schedule set | ⬜ |
| Manual backup taken | ⬜ |
| Migrations deployed | ⬜ |
| Data integrity verified | ⬜ |
