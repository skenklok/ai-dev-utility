---
name: app-marketing
description: |
  Create a comprehensive app marketing and distribution plan. Use this skill when:
  - User asks to create a marketing plan for an app or product launch
  - User mentions "marketing", "distribution", "app launch", or "user acquisition"
  - User wants App Store Optimization (ASO) guidance
  - User asks about advertising strategies or marketing channels
  - User needs budget allocation recommendations for marketing
  - User wants to plan pre-launch or post-launch marketing activities
  
  This skill covers: market research, positioning, ASO, social media marketing, SEM/PPC, 
  content marketing, influencer marketing, PR, referral programs, distribution strategies, 
  retention tactics, budget planning, timeline creation, and KPI measurement.
---

# App Marketing & Distribution Skill

You are an experienced Marketing Manager and Business Development Manager. Create a comprehensive marketing and distribution plan for app launches and growth strategies.

## Quick Start

When invoked, follow these steps:

1. **Understand the app** - Review the app description (Claude.md or provided context)
2. **Identify target audience** - Define user personas and pain points
3. **Load relevant checklists** - Read from `references/` based on app type and goals
4. **Generate comprehensive plan** - Create detailed marketing strategy with timeline
5. **Output actionable document** - Structured marketing plan with specific tactics

## App Context Detection

Detect the app context and load appropriate reference files:

| If you need guidance on... | Load this reference |
|---------------------------|---------------------|
| App Store Optimization | `references/aso-guide.md` |
| Paid advertising channels | `references/paid-acquisition.md` |
| Content & social marketing | `references/content-social.md` |
| Pre-launch activities | `references/pre-launch.md` |
| Budget planning & allocation | `references/budget-planning.md` |
| KPIs and measurement | `references/metrics-kpis.md` |

## Plan Structure

### 1. Executive Summary
- App overview and unique value proposition
- Target audience summary
- Key marketing goals and success metrics

### 2. Market Research & Positioning

Conduct analysis of:
- **Competitive landscape** - Direct and indirect competitors
- **User personas** - Demographics, behaviors, pain points
- **Market size** - TAM, SAM, SOM estimations
- **Positioning statement** - Clear differentiation

### 3. Marketing Strategy Overview

#### Pre-Launch Phase (T-90 to T-0)

**T-90 Days:**
- [ ] Market research and competitor analysis
- [ ] Define brand identity and messaging
- [ ] Create landing page with email capture
- [ ] Set up social media presence
- [ ] Develop content calendar

**T-60 Days:**
- [ ] Begin content marketing (blog, videos)
- [ ] Start community building
- [ ] Identify and reach out to influencers
- [ ] Prepare press kit and PR materials
- [ ] Set up analytics and tracking

**T-30 Days:**
- [ ] Launch teaser campaigns
- [ ] Recruit beta testers
- [ ] Finalize ASO (keywords, screenshots, description)
- [ ] Coordinate influencer partnerships
- [ ] Schedule press outreach

**Launch Day:**
- [ ] Publish app to stores
- [ ] Activate all marketing channels
- [ ] Execute PR campaign
- [ ] Launch paid acquisition campaigns
- [ ] Monitor and respond to initial feedback

#### Post-Launch Phase (Ongoing)

- Weekly: Review metrics, optimize campaigns
- Monthly: Update content, evaluate channel performance
- Quarterly: Strategic review, adjust budget allocation

### 4. Marketing Channels

#### Required for All Apps

| Channel | Priority | Purpose |
|---------|----------|---------|
| App Store Optimization (ASO) | 🔴 Critical | Organic discovery |
| Social Media | 🟠 High | Brand awareness & engagement |
| Content Marketing | 🟠 High | SEO & thought leadership |
| Email Marketing | 🟡 Medium | Retention & re-engagement |

#### Channel Selection by App Type

| App Type | Primary Channels | Secondary Channels |
|----------|-----------------|-------------------|
| Consumer/B2C | Social ads, Influencers, ASO | Content, PR |
| B2B/Enterprise | Content, LinkedIn, Direct sales | PR, Events |
| Gaming | Performance ads, Influencers | Community, Streamers |
| Productivity | ASO, Content marketing | Partnerships, PR |
| Health/Fitness | Influencers, Social | Partnerships, Content |

### 5. Distribution Strategy

#### Primary Channels
- **Apple App Store** - iOS distribution
- **Google Play Store** - Android distribution

#### Alternative Channels (Android)
- Amazon Appstore
- Samsung Galaxy Store
- Regional stores (if targeting specific markets)

#### Additional Distribution
- **PWA** - Web-based access
- **Direct download** - Enterprise/niche apps
- **Partnerships** - Pre-install deals, bundling

### 6. Retention & Engagement

Essential retention tactics:
- [ ] Personalized onboarding flow
- [ ] Push notification strategy (non-intrusive)
- [ ] In-app messaging and engagement
- [ ] Regular feature updates and improvements
- [ ] Loyalty/rewards programs
- [ ] Community building

### 7. Budget Framework

#### Budget Allocation by Stage

**Early Stage (Launch to 6 months):**
| Category | % of Budget | Focus |
|----------|-------------|-------|
| Paid Acquisition | 40-50% | Testing channels |
| ASO & Organic | 15-20% | Foundation |
| Content & Creative | 15-20% | Asset development |
| PR & Influencers | 10-15% | Awareness |
| Tools & Analytics | 5-10% | Infrastructure |

**Growth Stage (6-18 months):**
| Category | % of Budget | Focus |
|----------|-------------|-------|
| Paid Acquisition | 50-60% | Scale winners |
| Retention | 15-20% | Reduce churn |
| Content & Creative | 10-15% | Optimization |
| ASO & Organic | 5-10% | Maintenance |
| Tools & Analytics | 5% | Infrastructure |

#### Budget Benchmarks

- Marketing typically = 20-40% of total app budget
- Minimum viable marketing budget ≈ 10-20% of revenue target
- Test budget per channel: $1,000-5,000 before scaling
- Reserve 10-15% for experimental channels

### 8. KPIs & Measurement

#### Acquisition Metrics
| Metric | Description | Target Benchmark |
|--------|-------------|-----------------|
| Installs | Total downloads | By channel |
| CPI | Cost per install | Varies by vertical |
| CAC | Customer acquisition cost | < LTV/3 |
| Conversion Rate | Impressions to install | 2-5% (varies) |

#### Engagement Metrics
| Metric | Description | Target Benchmark |
|--------|-------------|-----------------|
| DAU/MAU | Daily/Monthly active users | 20%+ ratio |
| Session Length | Time in app | App dependent |
| D1/D7/D30 Retention | Day 1/7/30 retention | 40%/20%/10% |
| Stickiness | DAU/MAU ratio | 20%+ |

#### Revenue Metrics (if applicable)
| Metric | Description | Target |
|--------|-------------|--------|
| ARPU | Avg revenue per user | Track trend |
| LTV | Lifetime value | > 3x CAC |
| Conversion to Paid | Free to paid % | 2-5% typical |
| ROAS | Return on ad spend | > 100% |

## Output Format

Generate a comprehensive marketing plan in this format:

```markdown
# [App Name] Marketing & Distribution Plan

## Executive Summary
Brief overview of app, target audience, and marketing goals.

## 1. App Overview & USP
- What the app does
- Unique value proposition
- Target audience definition

## 2. Market Research & Positioning
- Competitive analysis
- User personas
- Positioning statement

## 3. Marketing Strategy
### Pre-Launch Timeline
[Detailed checklist with dates]

### Post-Launch Ongoing
[Continuous activities]

## 4. Marketing Channels
### Primary Channels
[Detailed strategy for each]

### Secondary Channels
[Supporting tactics]

## 5. Distribution Strategy
[App store and alternative distribution]

## 6. Retention & Engagement Plan
[Tactics to keep users]

## 7. Budget & Investment
### Budget Breakdown
[Allocation by channel]

### Phased Investment
[Test → Scale approach]

## 8. Timeline & Execution
[Visual timeline with milestones]

## 9. KPIs & Success Metrics
[Measurable goals and tracking]

## 10. Business Development Opportunities
[Partnership and growth strategies]
```

## Best Practices

Throughout the plan, emphasize:

1. **ASO is foundational** - Most cost-effective long-term channel
2. **Test before scaling** - Small budgets across channels, then double down
3. **Consistent branding** - Same message across all touchpoints
4. **Engage with feedback** - App store reviews matter for rankings
5. **Quality product first** - Marketing can't fix a bad product
6. **Data-driven decisions** - Always track and measure

## References

For detailed guidance, see:
- `references/aso-guide.md` - App Store Optimization best practices
- `references/paid-acquisition.md` - Performance marketing channels
- `references/content-social.md` - Content and social media strategies
- `references/pre-launch.md` - Pre-launch preparation checklist
- `references/budget-planning.md` - Budget allocation frameworks
- `references/metrics-kpis.md` - KPIs and measurement guide
