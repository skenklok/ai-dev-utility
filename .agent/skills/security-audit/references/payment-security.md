# Payment & Subscription Security Checklist

Security checks for Stripe, RevenueCat, and In-App Purchases (Apple/Google).

## Table of Contents
1. [Server-Side Validation (CRITICAL)](#server-side-validation-critical)
2. [RevenueCat Security](#revenuecat-security)
3. [Stripe Security](#stripe-security)
4. [Webhook Security](#webhook-security)
5. [Subscription Edge Cases](#subscription-edge-cases)

---

## Server-Side Validation (CRITICAL)

### The #1 Payment Security Rule

**NEVER trust the client for purchase validation. All receipts and entitlements MUST be verified server-side.**

Check for:
- [ ] Client-side receipt validation (CRITICAL vulnerability!)
- [ ] Entitlement checks only on client
- [ ] Premium features unlocked without server verification

**Vulnerable patterns:**
```javascript
// ❌ CRITICAL: Client-only validation
const purchase = await Purchases.purchaseProduct('premium');
if (purchase.customerInfo.entitlements.active['premium']) {
  unlockPremiumFeatures(); // No server verification!
}

// ❌ CRITICAL: Directly trusting StoreKit/Play Billing
const subscription = await storeKit.getProducts(['premium']);
if (subscription.isActive) {
  setIsPremium(true); // Client can be manipulated!
}
```

**Secure patterns:**
```javascript
// ✅ Server-side verification
const purchase = await Purchases.purchaseProduct('premium');

// Send to your server for verification
const response = await fetch('/api/verify-purchase', {
  method: 'POST',
  body: JSON.stringify({
    userId: currentUser.id,
    receiptData: purchase.transaction.transactionIdentifier
  })
});

const { verified, entitlements } = await response.json();
if (verified) {
  unlockPremiumFeatures();
}
```

---

## RevenueCat Security

### API Key Scoping

Check for:
- [ ] Secret API key exposed to client (should only be on server)
- [ ] Public key used where secret is required
- [ ] API keys in client-side code or bundles

**Key types:**
| Key Type | Use Case | Client-Safe? |
|----------|----------|--------------|
| Public API Key | Mobile SDK initialization | ✅ Yes |
| Secret API Key | Server-to-server calls, webhooks | ❌ Never |

**Vulnerable patterns:**
```javascript
// ❌ CRITICAL: Secret key in client code
Purchases.configure({
  apiKey: "appl_XXXXXXXXXXXXXXXX" // If this is a secret key, it's exposed!
});

// ❌ Secret key in .env bundled with app
const RC_SECRET = process.env.REVENUECAT_SECRET_KEY;
```

**Secure patterns:**
```javascript
// ✅ Public key for client SDK
Purchases.configure({
  apiKey: Platform.OS === 'ios' 
    ? "appl_PUBLIC_KEY_HERE" 
    : "goog_PUBLIC_KEY_HERE"
});

// ✅ Secret key only on server
// backend/services/revenuecat.ts
const response = await fetch('https://api.revenuecat.com/v1/subscribers/xxx', {
  headers: {
    'Authorization': `Bearer ${process.env.REVENUECAT_SECRET_KEY}` // Server env only
  }
});
```

### User ID Spoofing Prevention

Check for:
- [ ] User identity not verified server-side before granting entitlements

```javascript
// ❌ Trusting user ID from client
app.post('/api/check-subscription', async (req, res) => {
  const { userId } = req.body; // Can be spoofed!
  const subscriber = await revenueCat.getSubscriber(userId);
  return res.json({ isPremium: subscriber.entitlements.active['premium'] });
});

// ✅ Use authenticated user ID
app.post('/api/check-subscription', authMiddleware, async (req, res) => {
  const userId = req.user.id; // From verified JWT/session
  const subscriber = await revenueCat.getSubscriber(userId);
  return res.json({ isPremium: subscriber.entitlements.active['premium'] });
});
```

---

## Stripe Security

### API Key Exposure

Check for:
- [ ] Secret API key exposed to client
- [ ] Secret key in frontend code or bundled .env

**Search commands:**
```bash
# Find Stripe keys
grep -r "sk_live\|sk_test\|rk_live\|rk_test" --include="*.ts" --include="*.js"

# Check these are ONLY in server files
# If found in: /app/, /src/, mobile code = CRITICAL
```

### Payment Intent Verification

Check for:
- [ ] Payment intents created client-side
- [ ] Fulfillment based on `success_url` redirect (can be spoofed!)
- [ ] Missing webhook verification for order fulfillment

**Vulnerable patterns:**
```javascript
// ❌ CRITICAL: Success URL for fulfillment
app.get('/payment-success', (req, res) => {
  // User can directly access this URL!
  grantPurchasedItem(req.query.itemId);
});

// ❌ Trusting client-side payment confirmation
const paymentIntent = await stripe.confirmCardPayment(clientSecret);
if (paymentIntent.status === 'succeeded') {
  unlockContent(); // Client-side confirmation is untrusted!
}
```

**Secure patterns:**
```javascript
// ✅ Webhook for fulfillment
app.post('/webhooks/stripe', express.raw({ type: 'application/json' }), (req, res) => {
  const sig = req.headers['stripe-signature'];
  const event = stripe.webhooks.constructEvent(req.body, sig, webhookSecret);
  
  if (event.type === 'payment_intent.succeeded') {
    const paymentIntent = event.data.object;
    fulfillOrder(paymentIntent.metadata.orderId);
  }
  
  res.json({ received: true });
});
```

---

## Webhook Security

### Signature Verification (CRITICAL)

Check for:
- [ ] Stripe webhook without signature verification
- [ ] RevenueCat webhook without signature verification
- [ ] App Store Server Notifications without verification

**Vulnerable patterns:**
```javascript
// ❌ CRITICAL: No signature verification
app.post('/webhooks/stripe', async (req, res) => {
  const event = req.body; // Directly trusting request body!
  processPayment(event);
});

// ❌ CRITICAL: RevenueCat without auth
app.post('/webhooks/revenuecat', async (req, res) => {
  const event = req.body;
  updateSubscription(event);
});
```

**Secure patterns:**
```javascript
// ✅ Stripe with signature verification
app.post('/webhooks/stripe', 
  express.raw({ type: 'application/json' }),
  async (req, res) => {
    const sig = req.headers['stripe-signature'];
    try {
      const event = stripe.webhooks.constructEvent(
        req.body,
        sig,
        process.env.STRIPE_WEBHOOK_SECRET
      );
      // Process verified event
    } catch (err) {
      return res.status(400).send(`Webhook Error: ${err.message}`);
    }
  }
);

// ✅ RevenueCat with authorization header
app.post('/webhooks/revenuecat', async (req, res) => {
  const authHeader = req.headers['authorization'];
  if (authHeader !== `Bearer ${process.env.REVENUECAT_WEBHOOK_SECRET}`) {
    return res.status(401).send('Unauthorized');
  }
  // Process verified event
});
```

---

## Subscription Edge Cases

### Race Conditions

Check for:
- [ ] Race condition during purchase flow
- [ ] Subscription status not refreshed before granting access

```javascript
// ❌ Race condition - checking stale cache
if (cachedSubscriptionStatus.isPremium) {
  grantAccess();
}

// ✅ Always fetch fresh status for sensitive operations
const freshStatus = await revenueCat.getSubscriberInfo(userId);
if (freshStatus.entitlements.active['premium']) {
  grantAccess();
}
```

### Refund & Cancellation Handling

Check for:
- [ ] Refunds don't revoke access
- [ ] Cancellation doesn't update entitlements

```javascript
// ✅ Handle refund webhooks
if (event.type === 'CANCELLATION' || event.type === 'BILLING_ISSUE') {
  await revokeEntitlements(event.subscriber_id);
  await notifyUserOfRevocation(event.subscriber_id);
}
```

---

## Quick Reference Table

| Check | Severity | Files to Inspect |
|-------|----------|------------------|
| Client-only receipt validation | 🔴 Critical | Purchase flows, auth gates |
| Secret API key in client | 🔴 Critical | All client-side code |
| Missing webhook signatures | 🔴 Critical | Webhook handlers |
| success_url for fulfillment | 🔴 Critical | Payment completion flows |
| User ID spoofing | 🟠 High | API endpoints |
| Missing refund handling | 🟠 High | Webhook handlers |
| Race conditions | 🟡 Medium | Purchase flows |
