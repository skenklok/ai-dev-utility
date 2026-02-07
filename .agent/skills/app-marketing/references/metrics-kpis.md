# Marketing KPIs & Metrics Guide

Key performance indicators and measurement framework for app marketing.

## Table of Contents
1. [Acquisition Metrics](#acquisition-metrics)
2. [Engagement Metrics](#engagement-metrics)
3. [Revenue Metrics](#revenue-metrics)
4. [Channel-Specific KPIs](#channel-specific-kpis)
5. [Tracking Setup](#tracking-setup)

---

## Acquisition Metrics

### Core Acquisition KPIs

| Metric | Definition | Target |
|--------|------------|--------|
| Installs | Total app downloads | Growth trend |
| CPI | Cost per install | Below LTV/3 |
| CAC | Customer acquisition cost | Below LTV/3 |
| CTR | Click-through rate | 1-3% ads |
| CVR | Install conversion rate | 20-40% |
| IPM | Installs per 1000 impressions | 10+ |

### CPI Benchmarks by Category

| Category | iOS | Android |
|----------|-----|---------|
| Gaming | $1.50-4.00 | $0.75-2.50 |
| E-commerce | $2.00-5.00 | $1.00-3.00 |
| Finance | $5.00-15.00 | $3.00-8.00 |
| Social | $1.00-3.00 | $0.50-2.00 |
| Health/Fitness | $2.00-4.00 | $1.00-2.50 |
| Productivity | $1.50-3.00 | $0.75-2.00 |

---

## Engagement Metrics

### Retention Benchmarks

| Timeframe | Good | Excellent | Industry Avg |
|-----------|------|-----------|--------------|
| Day 1 | 30%+ | 40%+ | 25% |
| Day 7 | 15%+ | 20%+ | 12% |
| Day 30 | 8%+ | 12%+ | 4% |
| Day 90 | 4%+ | 6%+ | 2% |

### Engagement KPIs

| Metric | Definition | What It Tells You |
|--------|------------|-------------------|
| DAU | Daily active users | Daily engagement |
| WAU | Weekly active users | Weekly habits |
| MAU | Monthly active users | Monthly reach |
| DAU/MAU | Stickiness ratio | Product stickiness |
| Session Length | Time per session | Depth of use |
| Sessions/Day | Sessions per user | Frequency |

**Stickiness Benchmarks:**
| Ratio | Quality |
|-------|---------|
| 10-15% | Low |
| 15-20% | Average |
| 20-30% | Good |
| 30%+ | Excellent |

---

## Revenue Metrics

### Core Revenue KPIs

| Metric | Definition | Calculation |
|--------|------------|-------------|
| LTV | Lifetime value | Total revenue / Users |
| ARPU | Avg revenue per user | Revenue / All users |
| ARPPU | Avg revenue per paying user | Revenue / Paying users |
| Conversion Rate | Free to paid % | Paid users / Total users |
| ROAS | Return on ad spend | Revenue / Ad spend |

### LTV Calculation Methods

**Simple LTV:**
```
LTV = ARPU × Average Lifespan (months)
```

**Subscription LTV:**
```
LTV = ARPU × (1 / Churn Rate)
```

**LTV:CAC Targets:**
| Ratio | Interpretation |
|-------|----------------|
| <1:1 | Losing money |
| 1:1-2:1 | Break-even |
| 2:1-3:1 | Healthy |
| 3:1+ | Efficient |

### Subscription Metrics

| Metric | Definition | Target |
|--------|------------|--------|
| Trial Conversion | Trial → Paid | 30-60% |
| Churn Rate | Monthly cancellations | <5% |
| MRR | Monthly recurring revenue | Growth |
| Net Revenue Retention | Existing revenue growth | >100% |

---

## Channel-Specific KPIs

### Paid Acquisition

| Channel | Primary KPI | Secondary KPIs |
|---------|-------------|----------------|
| Apple Search Ads | CPI, ROAS | TTR, CVR |
| Google UAC | CPI, CPA | ROAS, Retention |
| Meta Ads | CPI, CPM | CTR, Frequency |
| TikTok | CPI, VTR | Engagement rate |

### Organic Channels

| Channel | Primary KPI | Secondary KPIs |
|---------|-------------|----------------|
| ASO | Keyword rankings | Impressions, CVR |
| Content/SEO | Organic traffic | Time on page, bounces |
| Social | Followers, engagement | Reach, shares |
| Referral | Referral rate | Viral coefficient |

### PR/Influencer

| Metric | Definition |
|--------|------------|
| Media impressions | Total reach |
| Share of voice | Brand mentions vs. competitors |
| Referral traffic | Visits from coverage |
| Backlinks | SEO value |
| Cost per engagement | $ / engagement |

---

## Tracking Setup

### Essential Analytics Tools

| Purpose | Tools |
|---------|-------|
| Attribution (MMP) | AppsFlyer, Adjust, Branch |
| App Analytics | Firebase, Amplitude, Mixpanel |
| Web Analytics | Google Analytics 4 |
| A/B Testing | Optimizely, LaunchDarkly |

### Attribution Setup Checklist

- [ ] Choose and implement MMP SDK
- [ ] Configure deep linking
- [ ] Set up postbacks to ad networks
- [ ] Define conversion events
- [ ] Configure SKAdNetwork (iOS)
- [ ] Test attribution flows

### Event Tracking Fundamentals

**Essential Events:**
| Event | Why Track |
|-------|-----------|
| app_open | Active users |
| registration | Conversion funnel |
| onboarding_complete | Activation |
| first_purchase | Monetization |
| subscription_start | Revenue |
| key_feature_used | Engagement |

---

## Dashboard Essentials

### Daily Monitoring

| Metric | Alert Threshold |
|--------|----------------|
| Installs | -30% vs. avg |
| CPI | +50% vs. target |
| Day 1 Retention | -10% vs. avg |
| Crash Rate | >1% |
| Revenue | -20% vs. avg |

### Weekly Review

1. Channel performance (ROAS, CPI)
2. Retention cohorts
3. A/B test results
4. Creative performance
5. ASO rankings

### Monthly Analysis

1. LTV by channel/cohort
2. CAC payback trends
3. Churn analysis
4. Budget reallocation
5. Competitive review

---

## Quick Reference: Metric Priorities by Stage

| Stage | Focus Metrics |
|-------|---------------|
| Pre-launch | Waitlist signups, landing CVR |
| Launch | Installs, D1/D7 retention, CPI |
| Growth | LTV, CAC, ROAS, D30 retention |
| Mature | Churn, NRR, LTV:CAC, profitability |
