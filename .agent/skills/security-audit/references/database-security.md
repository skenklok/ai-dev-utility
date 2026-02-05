# Database Security Checklist (Prisma + PostgreSQL)

Security checks for Prisma ORM and PostgreSQL databases.

## Table of Contents
1. [Injection Prevention](#injection-prevention)
2. [IDOR Prevention](#idor-prevention)
3. [Connection Security](#connection-security)
4. [Data Protection](#data-protection)
5. [Mass Assignment](#mass-assignment)

---

## Injection Prevention

### Raw Query Vulnerabilities

Prisma's type-safe query builder prevents most SQL injection, but raw queries are dangerous.

Check for:
- [ ] `$executeRaw` with string interpolation
- [ ] `$queryRaw` with unsanitized input
- [ ] `$executeRawUnsafe` / `$queryRawUnsafe` usage

**Search commands:**
```bash
# Find all raw query usage
grep -rn "\$executeRaw\|\$queryRaw\|\$executeRawUnsafe\|\$queryRawUnsafe" --include="*.ts"
```

**Vulnerable patterns:**
```typescript
// ❌ CRITICAL: SQL injection via template literal
const userId = req.params.id; // User input
const user = await prisma.$queryRaw`SELECT * FROM users WHERE id = ${userId}`;

// ❌ CRITICAL: String concatenation
const query = `SELECT * FROM users WHERE name = '${userName}'`;
await prisma.$executeRawUnsafe(query);

// ❌ CRITICAL: Unsafe methods
await prisma.$queryRawUnsafe(`SELECT * FROM products WHERE category = '${category}'`);
```

**Secure patterns:**
```typescript
// ✅ Use Prisma's type-safe queries
const user = await prisma.user.findUnique({
  where: { id: userId }
});

// ✅ If raw query needed, use Prisma.sql for parameterization
import { Prisma } from '@prisma/client';
const users = await prisma.$queryRaw(
  Prisma.sql`SELECT * FROM users WHERE id = ${userId}`
);

// ✅ Or use $queryRaw with proper tagged template
const users = await prisma.$queryRaw`
  SELECT * FROM users WHERE email = ${email}::text
`;
```

---

## IDOR Prevention

### Insecure Direct Object Reference

Check for:
- [ ] Queries that don't filter by authenticated user
- [ ] Direct access to resources by ID without ownership check
- [ ] Missing authorization middleware

**Vulnerable patterns:**
```typescript
// ❌ IDOR: Any user can access any order
app.get('/api/orders/:orderId', async (req, res) => {
  const order = await prisma.order.findUnique({
    where: { id: req.params.orderId }
    // Missing: check if order belongs to authenticated user!
  });
  return res.json(order);
});

// ❌ IDOR: User can modify others' profiles
app.put('/api/users/:userId', async (req, res) => {
  await prisma.user.update({
    where: { id: req.params.userId }, // No ownership check!
    data: req.body
  });
});
```

**Secure patterns:**
```typescript
// ✅ Always include user context in queries
app.get('/api/orders/:orderId', authMiddleware, async (req, res) => {
  const order = await prisma.order.findFirst({
    where: {
      id: req.params.orderId,
      userId: req.user.id  // Only user's own orders
    }
  });
  
  if (!order) {
    return res.status(404).json({ error: 'Order not found' });
  }
  
  return res.json(order);
});

// ✅ For admin endpoints, verify admin role
app.get('/api/admin/users/:userId', authMiddleware, async (req, res) => {
  if (req.user.role !== 'ADMIN') {
    return res.status(403).json({ error: 'Forbidden' });
  }
  // Admin can access any user
});
```

### Multi-Tenant Data Isolation

Check for:
- [ ] Queries missing tenant/organization filter
- [ ] Cross-tenant data access possible

```typescript
// ❌ Missing tenant filter
const projects = await prisma.project.findMany();

// ✅ Include tenant context
const projects = await prisma.project.findMany({
  where: { organizationId: req.user.organizationId }
});
```

---

## Connection Security

### Secure Database Connection

Check for:
- [ ] DATABASE_URL logged or exposed in errors
- [ ] SSL not enforced for production
- [ ] Connection string committed to git

**Search commands:**
```bash
# Check for exposed connection strings
grep -rn "DATABASE_URL\|postgresql://\|postgres://" --include="*.ts" --include="*.js" --include="*.env"

# Verify .env is gitignored
cat .gitignore | grep ".env"
```

**Vulnerable patterns:**
```typescript
// ❌ Logging connection string
console.log('Connecting to:', process.env.DATABASE_URL);

// ❌ Exposing in error response
catch (error) {
  res.status(500).json({ 
    error: error.message,
    connectionString: process.env.DATABASE_URL // Exposed!
  });
}
```

**Secure patterns:**
```typescript
// ✅ Never log or expose connection details
try {
  await prisma.$connect();
} catch (error) {
  console.error('Database connection failed'); // No details
  throw new Error('Service temporarily unavailable');
}
```

**Check DATABASE_URL has SSL:**
```bash
# Should include sslmode=require
DATABASE_URL="postgresql://user:pass@host:5432/db?sslmode=require"
```

---

## Data Protection

### Sensitive Data Handling

Check for:
- [ ] Passwords stored as plaintext
- [ ] PII not encrypted at rest
- [ ] Sensitive data in query results not filtered

**Vulnerable patterns:**
```typescript
// ❌ CRITICAL: Plaintext password
await prisma.user.create({
  data: {
    email: 'user@example.com',
    password: 'plaintextpassword123' // Never do this!
  }
});

// ❌ Returning password hash to client
const user = await prisma.user.findUnique({ where: { id: userId } });
res.json(user); // Includes password field!
```

**Secure patterns:**
```typescript
// ✅ Hash passwords with bcrypt
import bcrypt from 'bcrypt';
const hashedPassword = await bcrypt.hash(password, 12);
await prisma.user.create({
  data: {
    email: 'user@example.com',
    password: hashedPassword
  }
});

// ✅ Select only needed fields
const user = await prisma.user.findUnique({
  where: { id: userId },
  select: {
    id: true,
    email: true,
    name: true
    // password explicitly excluded
  }
});
```

### Soft Delete Bypass

Check for:
- [ ] Soft-deleted records accessible through queries

```typescript
// ❌ Soft-deleted records accessible
const users = await prisma.user.findMany(); // Includes deleted!

// ✅ Filter out deleted records
const users = await prisma.user.findMany({
  where: { deletedAt: null }
});

// ✅ Or use Prisma middleware for automatic filtering
prisma.$use(async (params, next) => {
  if (params.model === 'User' && params.action === 'findMany') {
    params.args.where = { ...params.args.where, deletedAt: null };
  }
  return next(params);
});
```

---

## Mass Assignment

### Prevent Uncontrolled Data Updates

Check for:
- [ ] Request body passed directly to create/update
- [ ] Users able to set admin/role fields

**Vulnerable patterns:**
```typescript
// ❌ CRITICAL: Mass assignment
app.post('/api/users', async (req, res) => {
  const user = await prisma.user.create({
    data: req.body // User could set { role: 'ADMIN' }!
  });
});

// ❌ Same for updates
app.put('/api/users/:id', async (req, res) => {
  await prisma.user.update({
    where: { id: req.params.id },
    data: req.body // Could escalate privileges
  });
});
```

**Secure patterns:**
```typescript
// ✅ Explicitly pick allowed fields
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
  password: z.string().min(8)
  // role intentionally excluded
});

app.post('/api/users', async (req, res) => {
  const data = CreateUserSchema.parse(req.body);
  const user = await prisma.user.create({
    data: {
      ...data,
      role: 'USER' // Explicitly set default
    }
  });
});
```

---

## Quick Reference Table

| Check | Severity | Files to Inspect |
|-------|----------|------------------|
| Raw queries with interpolation | 🔴 Critical | All Prisma usage |
| IDOR - missing user context | 🔴 Critical | API routes |
| Plaintext passwords | 🔴 Critical | Auth, user creation |
| Mass assignment | 🔴 Critical | Create/update endpoints |
| DATABASE_URL exposed | 🟠 High | Error handlers, logs |
| Missing SSL | 🟠 High | Database config |
| Soft delete bypass | 🟡 Medium | Query patterns |
| Over-fetching data | 🟡 Medium | Select statements |
