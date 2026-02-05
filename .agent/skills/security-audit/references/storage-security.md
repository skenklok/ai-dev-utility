# Storage Security Checklist (Cloudflare R2)

Security checks for Cloudflare R2 storage and S3-compatible object storage.

## Table of Contents
1. [Access Control](#access-control)
2. [Signed URLs](#signed-urls)
3. [Credential Security](#credential-security)
4. [Upload Security](#upload-security)
5. [Public Access](#public-access)

---

## Access Control

### Bucket Access Configuration

Check for:
- [ ] Bucket publicly accessible when it shouldn't be
- [ ] R2 credentials exposed to client
- [ ] Missing authentication on file access routes

**Search commands:**
```bash
# Find R2/S3 configuration
grep -rn "R2_\|CLOUDFLARE\|accountid.*r2\|S3Client\|@aws-sdk/client-s3" --include="*.ts" --include="*.js"

# Find bucket operations
grep -rn "PutObjectCommand\|GetObjectCommand\|putObject\|getObject" --include="*.ts" --include="*.js"
```

**Vulnerable patterns:**
```typescript
// ❌ CRITICAL: R2 credentials in client code
const r2Client = new S3Client({
  region: 'auto',
  endpoint: `https://${accountId}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: 'your-access-key',      // Exposed!
    secretAccessKey: 'your-secret-key'   // Exposed!
  }
});

// ❌ Directly exposing bucket URLs
app.get('/files/:key', async (req, res) => {
  const url = `https://bucket.r2.cloudflarestorage.com/${req.params.key}`;
  res.redirect(url); // May expose private files!
});
```

**Secure patterns:**
```typescript
// ✅ R2 client only on server with env vars
// backend/services/r2.ts
import { S3Client } from '@aws-sdk/client-s3';

export const r2Client = new S3Client({
  region: 'auto',
  endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!
  }
});

// ✅ Server-side file access with auth
app.get('/files/:key', authMiddleware, async (req, res) => {
  // Verify user owns this file
  const file = await prisma.file.findFirst({
    where: {
      key: req.params.key,
      userId: req.user.id
    }
  });
  
  if (!file) {
    return res.status(404).json({ error: 'File not found' });
  }
  
  // Generate signed URL for download
  const signedUrl = await getSignedUrl(r2Client, 
    new GetObjectCommand({
      Bucket: process.env.R2_BUCKET_NAME,
      Key: file.key
    }),
    { expiresIn: 3600 }
  );
  
  res.json({ url: signedUrl });
});
```

---

## Signed URLs

### Secure Pre-Signed URL Generation

Check for:
- [ ] Signed URLs with very long expiration
- [ ] No validation before generating download URLs
- [ ] Upload URLs generated without proper restrictions

**Vulnerable patterns:**
```typescript
// ❌ Very long expiration (1 year!)
const signedUrl = await getSignedUrl(r2Client, command, {
  expiresIn: 31536000
});

// ❌ No authorization before generating URL
app.get('/download/:fileId', async (req, res) => {
  // Anyone can download any file!
  const signedUrl = await generateDownloadUrl(req.params.fileId);
  res.json({ url: signedUrl });
});

// ❌ Upload URL without content type restriction
app.post('/upload-url', async (req, res) => {
  const signedUrl = await getSignedUrl(r2Client,
    new PutObjectCommand({
      Bucket: process.env.R2_BUCKET_NAME,
      Key: req.body.filename
    }),
    { expiresIn: 3600 }
  );
  res.json({ uploadUrl: signedUrl });
});
```

**Secure patterns:**
```typescript
// ✅ Short expiration times
const DOWNLOAD_URL_EXPIRY = 15 * 60;  // 15 minutes
const UPLOAD_URL_EXPIRY = 5 * 60;     // 5 minutes

// ✅ Validate authorization before generating download URL
app.get('/download/:fileId', authMiddleware, async (req, res) => {
  const file = await prisma.file.findFirst({
    where: {
      id: req.params.fileId,
      userId: req.user.id
    }
  });
  
  if (!file) {
    return res.status(404).json({ error: 'Not found' });
  }
  
  const signedUrl = await getSignedUrl(r2Client,
    new GetObjectCommand({
      Bucket: process.env.R2_BUCKET_NAME,
      Key: file.key
    }),
    { expiresIn: DOWNLOAD_URL_EXPIRY }
  );
  
  res.json({ url: signedUrl });
});

// ✅ Upload URL with content restrictions
app.post('/upload-url', authMiddleware, async (req, res) => {
  const { filename, contentType, fileSize } = req.body;
  
  // Validate content type
  const allowedTypes = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf'];
  if (!allowedTypes.includes(contentType)) {
    return res.status(400).json({ error: 'Invalid file type' });
  }
  
  // Validate file size
  const MAX_SIZE = 10 * 1024 * 1024; // 10MB
  if (fileSize > MAX_SIZE) {
    return res.status(400).json({ error: 'File too large' });
  }
  
  // Generate unique key
  const key = `${req.user.id}/${uuidv4()}-${sanitizeFilename(filename)}`;
  
  const signedUrl = await getSignedUrl(r2Client,
    new PutObjectCommand({
      Bucket: process.env.R2_BUCKET_NAME,
      Key: key,
      ContentType: contentType,
      ContentLength: fileSize
    }),
    { expiresIn: UPLOAD_URL_EXPIRY }
  );
  
  // Record pending upload
  await prisma.file.create({
    data: {
      key,
      userId: req.user.id,
      contentType,
      status: 'pending'
    }
  });
  
  res.json({ uploadUrl: signedUrl, key });
});
```

---

## Credential Security

### R2 API Token Management

Check for:
- [ ] R2 credentials in code or committed to git
- [ ] Same credentials for all environments
- [ ] Overly permissive token scopes

**Cloudflare R2 Token Scopes:**
| Scope | Use Case |
|-------|----------|
| Object Read | Download files |
| Object Read & Write | Upload + download |
| Admin Read & Write | Full bucket access |

**Vulnerable patterns:**
```typescript
// ❌ Credentials committed to code
const R2_ACCESS_KEY = 'xxxxxxxxxxxxxxxxxxxx';
const R2_SECRET_KEY = 'yyyyyyyyyyyyyyyyyyyy';

// ❌ Same credentials in .env.development and .env.production
```

**Secure patterns:**
```typescript
// ✅ Use Heroku Config Vars / environment variables
// Never commit these values

// .env.example (commit this, not .env)
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=
CLOUDFLARE_ACCOUNT_ID=

// ✅ Validate credentials exist at startup
const requiredEnvVars = [
  'R2_ACCESS_KEY_ID',
  'R2_SECRET_ACCESS_KEY',
  'R2_BUCKET_NAME',
  'CLOUDFLARE_ACCOUNT_ID'
];

for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    throw new Error(`Missing required environment variable: ${envVar}`);
  }
}
```

---

## Upload Security

### Secure File Upload Handling

Check for:
- [ ] No file type validation
- [ ] No file size limits
- [ ] Filename not sanitized (path traversal)
- [ ] Uploaded malware not scanned

**Vulnerable patterns:**
```typescript
// ❌ No validation on upload
app.post('/upload', async (req, res) => {
  const { filename, data } = req.body;
  await r2Client.send(new PutObjectCommand({
    Bucket: bucket,
    Key: filename,        // Path traversal: "../../../etc/passwd"
    Body: Buffer.from(data, 'base64')
  }));
});

// ❌ Trusting Content-Type header
app.post('/upload', upload.single('file'), async (req, res) => {
  // Attacker can set Content-Type: image/jpeg on a .exe file
  await uploadToR2(req.file.buffer, req.file.mimetype);
});
```

**Secure patterns:**
```typescript
import { fileTypeFromBuffer } from 'file-type';
import sanitize from 'sanitize-filename';

// ✅ Comprehensive upload validation
app.post('/upload', authMiddleware, upload.single('file'), async (req, res) => {
  const file = req.file;
  
  if (!file) {
    return res.status(400).json({ error: 'No file provided' });
  }
  
  // Validate file size
  const MAX_SIZE = 10 * 1024 * 1024;
  if (file.size > MAX_SIZE) {
    return res.status(400).json({ error: 'File too large' });
  }
  
  // Detect actual file type from magic bytes (not header)
  const detectedType = await fileTypeFromBuffer(file.buffer);
  const allowedTypes = ['image/jpeg', 'image/png', 'image/webp'];
  
  if (!detectedType || !allowedTypes.includes(detectedType.mime)) {
    return res.status(400).json({ error: 'Invalid file type' });
  }
  
  // Sanitize filename to prevent path traversal
  const sanitizedName = sanitize(file.originalname);
  const key = `uploads/${req.user.id}/${Date.now()}-${sanitizedName}`;
  
  await r2Client.send(new PutObjectCommand({
    Bucket: process.env.R2_BUCKET_NAME,
    Key: key,
    Body: file.buffer,
    ContentType: detectedType.mime
  }));
  
  res.json({ key });
});
```

---

## Public Access

### Cloudflare R2 Public Bucket Configuration

Check for:
- [ ] Private bucket exposed via public domain
- [ ] Sensitive files in public bucket
- [ ] Missing Cache-Control for sensitive files

**R2 access patterns:**

| Access Type | Use Case | Security |
|-------------|----------|----------|
| Private (signed URLs) | User uploads, sensitive docs | ✅ Recommended |
| Public (custom domain) | Static assets, public images | ⚠️ Use carefully |
| Workers (R2 binding) | Server-side access | ✅ Good |

**Verify bucket privacy:**
```bash
# Check if bucket has public access enabled
# In Cloudflare Dashboard: R2 > Bucket > Settings > Public access

# Check for custom domain attached (makes bucket public at that domain)
```

**Secure patterns for public assets:**
```typescript
// ✅ Separate buckets for public vs private
const privateBucket = process.env.R2_PRIVATE_BUCKET;  // User files
const publicBucket = process.env.R2_PUBLIC_BUCKET;    // Static assets

// ✅ For public bucket, use Cache-Control
await r2Client.send(new PutObjectCommand({
  Bucket: publicBucket,
  Key: `assets/${filename}`,
  Body: buffer,
  ContentType: 'image/png',
  CacheControl: 'public, max-age=31536000' // 1 year for static assets
}));

// ✅ For private bucket, always use signed URLs with short expiry
```

---

## Quick Reference Table

| Check | Severity | Files to Inspect |
|-------|----------|------------------|
| R2 credentials in client | 🔴 Critical | All client-side files |
| R2 credentials committed | 🔴 Critical | .git, .env files |
| No auth on file access | 🔴 Critical | File download routes |
| No file type validation | 🟠 High | Upload handlers |
| Long signed URL expiry | 🟠 High | URL generation code |
| No filename sanitization | 🟠 High | Upload handlers |
| Private bucket public exposed | 🟠 High | Cloudflare dashboard |
| Missing file size limits | 🟡 Medium | Upload handlers |
