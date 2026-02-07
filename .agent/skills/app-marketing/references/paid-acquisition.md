# Paid Acquisition Channels Guide

Comprehensive guide to paid user acquisition channels for mobile apps.

## Table of Contents
1. [Google Ads (UAC)](#google-ads-uac)
2. [Apple Search Ads](#apple-search-ads)
3. [Meta Ads (Facebook/Instagram)](#meta-ads-facebookinstagram)
4. [TikTok Ads](#tiktok-ads)
5. [Other Networks](#other-networks)
6. [Testing & Optimization](#testing--optimization)

---

## Google Ads (UAC)

### Overview
Google App Campaigns (formerly Universal App Campaigns) automatically optimize across:
- Google Search
- Google Play Store
- YouTube
- Display Network
- Discover

### Campaign Types

| Type | Goal | Best For |
|------|------|----------|
| Install | Drive downloads | Launch, top-of-funnel |
| In-App Actions | Post-install events | Monetization focus |
| Pre-registration (Android) | Pre-launch waitlist | Building anticipation |

### Setup Checklist

- [ ] Link Google Ads to Play Console
- [ ] Set up conversion tracking (Firebase or third-party MMP)
- [ ] Define in-app events to track
- [ ] Prepare ad assets:
  - 2-5 text headlines (30 chars each)
  - 1-5 descriptions (90 chars each)
  - Up to 20 images
  - Up to 20 videos (landscape, portrait, square)
  - HTML5 playable ads (optional)

### Bidding Strategies

| Strategy | Use When |
|----------|----------|
| Target CPI | Controlling install costs |
| Target CPA | Optimizing for specific events |
| ROAS | Revenue optimization (requires data) |

### Budget Recommendations

- **Minimum viable test**: $500-1,000/week
- **Scaling budget**: 10-20% increases per week max
- **Learning period**: 2-4 weeks for algorithm optimization
- **Target**: 50+ conversions/week for stable optimization

### Optimization Tips

1. **Creative refresh** - New assets every 2-4 weeks
2. **Audience layering** - Don't over-target initially
3. **Bid adjustments** - Gradual changes, monitor impact
4. **Geographic targeting** - Start with top-performing countries
5. **Time to conversion** - Consider longer attribution windows

---

## Apple Search Ads

### Campaign Types

| Type | Placement | Control | Best For |
|------|-----------|---------|----------|
| Basic | App Store only | Limited | Small budgets |
| Advanced | App Store | Full | Scale & optimization |

### Search Ads Advanced Structure

```
ACCOUNT
└── CAMPAIGN (budget, goal)
    └── AD GROUP (keywords, audience, CPT bid)
        └── CREATIVE SETS (custom screenshots)
```

### Keyword Strategy

**Match Types:**
| Match Type | Example | When to Use |
|------------|---------|-------------|
| Exact | [fitness app] | High-intent, controlled |
| Broad | fitness app | Discovery, expansion |

**Keyword Categories:**
1. **Brand terms** - Your app name (defense strategy)
2. **Category terms** - Generic (fitness, productivity)
3. **Competitor terms** - Other app names (expensive)
4. **Feature terms** - Specific features (workout tracker)

### Audience Targeting

| Audience | Description | Use Case |
|----------|-------------|----------|
| All Users | No targeting | Broad reach |
| New Users | Haven't downloaded | Acquisition |
| Returning | Downloaded, deleted | Re-engagement |
| Users of My Other Apps | Cross-sell | Portfolio strategy |

### Benchmarks (vary by category)

| Metric | Gaming | Non-Gaming |
|--------|--------|------------|
| Avg. CPI | $2-4 | $1-2 |
| Tap-Through Rate | 5-10% | 7-12% |
| Conversion Rate | 40-60% | 50-70% |

### Optimization Tips

- [ ] Bid on your own brand name (defensive)
- [ ] Use negative keywords to exclude irrelevant
- [ ] Create custom creative sets per ad group
- [ ] Monitor search term reports weekly
- [ ] Adjust bids based on performance data

---

## Meta Ads (Facebook/Instagram)

### Campaign Objectives

| Objective | Goal | Best For |
|-----------|------|----------|
| App Installs | Downloads | Acquisition |
| Conversions | In-app events | Quality users |
| Value Optimization | Revenue | Monetization |

### Campaign Structure

```
CAMPAIGN
└── AD SET (audience, budget, schedule)
    └── AD (creative, copy)
```

### Audience Types

| Type | Definition | Size | Performance |
|------|------------|------|-------------|
| Broad | Demographics only | Large | Good for discovery |
| Interest | Based on interests | Medium | Targeted |
| Lookalike | Similar to seed audience | Variable | High quality |
| Custom | Your own data | Variable | Retargeting |

### Creative Best Practices

**Video Ads (recommended):**
- 15-30 seconds optimal
- Vertical (9:16) for Stories/Reels
- Square (1:1) for Feed
- Hook in first 3 seconds
- Include captions (85% watch muted)

**Image Ads:**
- Minimal text (< 20% of image)
- Clear value proposition
- High contrast colors
- Mobile-first design

### Testing Framework

**Phase 1: Discovery**
- 3-5 different creative concepts
- Broad targeting
- Small budget ($20-50/day per ad set)

**Phase 2: Optimization**
- Scale winning creatives
- Test audience variations
- Increase budgets gradually

**Phase 3: Scale**
- Horizontal scaling (new ad sets)
- Vertical scaling (increase budgets)
- Refresh creatives to combat fatigue

### CPI Benchmarks

| Category | iOS | Android |
|----------|-----|---------|
| Gaming | $1.50-4.00 | $0.75-2.50 |
| E-commerce | $2.00-5.00 | $1.00-3.00 |
| Finance | $5.00-15.00 | $3.00-8.00 |
| Social | $1.00-3.00 | $0.50-2.00 |

---

## TikTok Ads

### Why TikTok

- Massive Gen Z and Millennial audience
- High engagement rates
- Lower CPIs than Meta (currently)
- Content-first algorithm

### Campaign Types

| Type | Goal |
|------|------|
| App Install | Drive downloads |
| App Event Optimization | In-app conversions |
| Value-Based Optimization | ROAS focus |

### Creative Guidelines

**Video Requirements:**
- 9:16 vertical (required)
- 9-15 seconds optimal
- Native, authentic feel
- Include trending sounds/music
- Direct call-to-action

**Creative Tips:**
1. **Look native** - Blend with organic content
2. **Use creators** - Partner with TikTok creators
3. **Fast pacing** - Quick cuts, dynamic content
4. **Trending formats** - Leverage trends appropriately
5. **Test volumes** - High creative refresh rate needed

### Targeting Options

- Demographics (age, gender, location)
- Interests and behaviors
- Custom audiences (first-party data)
- Lookalike audiences
- Automatic targeting (recommended for start)

### TikTok Spark Ads

Promote organic creator content as ads:
- Higher engagement than traditional ads
- Authentic feel
- Partner with creators for content
- Can use creator's profile

---

## Other Networks

### Unity Ads
- Best for: Gaming apps
- Rewarded video and interstitial
- High-quality gaming audience

### ironSource
- Best for: Mid-core and casual games
- Offerwalls, rewarded video
- Good for monetization + UA

### Vungle
- Best for: Performance campaigns
- High-quality video ads
- Creative tools included

### Snap Ads
- Best for: Young demographic
- AR features unique
- Lower CPIs, smaller scale

### X (Twitter) Ads
- Best for: Specific niches, news/finance apps
- Limited scale compared to Meta/Google
- Good for B2B or professional apps

### Reddit Ads
- Best for: Niche communities
- Engaged audiences in specific topics
- Authenticity matters

---

## Testing & Optimization

### Test-and-Scale Framework

```
PHASE 1: TEST ($1-5K)
├── Test 3-5 channels simultaneously
├── Small budgets per channel
├── Focus on install volume + quality signals
└── Timeline: 2-4 weeks

PHASE 2: OPTIMIZE ($5-20K)
├── Cut losing channels
├── Double down on winners
├── Test variations within channels
└── Timeline: 4-8 weeks

PHASE 3: SCALE ($20K+)
├── Significant budget to winning channels
├── Horizontal scaling (new audiences)
├── Creative refresh cycle
└── Ongoing optimization
```

### Key Metrics to Track

| Metric | Definition | Action Threshold |
|--------|------------|------------------|
| CPI | Cost per install | Compare to LTV |
| CPA | Cost per action (event) | Compare to value |
| ROAS | Revenue / Ad Spend | Target > 100% |
| Retention (D1/7/30) | Users returning | Industry benchmark |
| IPM | Installs per 1000 impressions | Creative quality |

### Attribution & Analytics

**Mobile Measurement Partners (MMPs):**
- AppsFlyer
- Adjust
- Branch
- Singular
- Kochava

**Key Attribution Features:**
- Last-click attribution
- View-through attribution (VTA)
- Multi-touch attribution (MTA)
- Fraud detection
- Deep linking
- SKAdNetwork support (iOS 14.5+)

### iOS 14.5+ Considerations

**Impact:**
- Limited targeting due to ATT
- SKAdNetwork for attribution
- Delayed and aggregated data
- Privacy-focused measurement

**Adaptation Strategies:**
- Probabilistic modeling
- Google's Privacy Sandbox
- First-party data importance
- Contextual targeting
- Aggregated attribution

---

## Budget Allocation Template

### For New App Launch

| Channel | % of Budget | Notes |
|---------|-------------|-------|
| Apple Search Ads | 25% | High intent, measurable |
| Google UAC | 25% | Scale potential |
| Meta (FB/IG) | 30% | Creative testing |
| TikTok | 15% | Emerging, lower CPIs |
| Testing Reserve | 5% | New channels |

### Adjust Based On

- App category (gaming vs. non-gaming)
- Target audience (age, platform preference)
- Budget size (smaller = more focus)
- Historical performance data
- LTV:CAC ratio per channel
