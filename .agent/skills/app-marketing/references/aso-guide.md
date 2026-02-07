# App Store Optimization (ASO) Guide

Comprehensive checklist and best practices for optimizing app store presence.

## Table of Contents
1. [Keyword Optimization](#keyword-optimization)
2. [Visual Assets](#visual-assets)
3. [App Description & Metadata](#app-description--metadata)
4. [Ratings & Reviews](#ratings--reviews)
5. [Localization](#localization)

---

## Keyword Optimization

### Research Process

- [ ] Identify core keywords (what users search for)
- [ ] Analyze competitor keywords (use tools like AppFollow, Sensor Tower)
- [ ] Consider long-tail keywords (less competition, more specific)
- [ ] Track keyword rankings over time

### Platform-Specific Guidelines

**Apple App Store:**
| Element | Character Limit | Best Practice |
|---------|-----------------|---------------|
| App Name | 30 chars | Primary keyword in name |
| Subtitle | 30 chars | Secondary keywords |
| Keyword Field | 100 chars | Comma-separated, no spaces |

**Google Play Store:**
| Element | Character Limit | Best Practice |
|---------|-----------------|---------------|
| App Title | 30 chars | Primary keyword in title |
| Short Description | 80 chars | Keywords + value prop |
| Long Description | 4000 chars | Natural keyword density (3-5%) |

### Keyword Selection Tips

1. **Relevance first** - Only use keywords that truly describe your app
2. **Search volume** - Prioritize keywords with traffic
3. **Competition** - Balance volume with difficulty
4. **User intent** - Match search intent (discovery vs. specific)
5. **Avoid branded terms** - Don't use competitor brand names

---

## Visual Assets

### App Icon

**Best Practices:**
- Simple, recognizable silhouette
- Limited color palette (2-3 colors max)
- No text (unreadable at small sizes)
- Test at multiple sizes (small icon used most)
- A/B test different variations

**Common Mistakes:**
- ❌ Too much detail
- ❌ Thin lines or small elements
- ❌ Text or words in icon
- ❌ Generic or stock-looking designs

### Screenshots

**Apple App Store Requirements:**
- Up to 10 screenshots per device type
- First 3 screenshots appear in search (iOS 11+)
- Sizes: iPhone 6.5", iPhone 5.5", iPad Pro

**Google Play Requirements:**
- 2-8 screenshots
- All device types supported
- Sizes: Various device dimensions

**Screenshot Best Practices:**

| Order | Content | Purpose |
|-------|---------|---------|
| 1 | Hero feature | First impression, key value |
| 2 | Core functionality | Primary use case |
| 3 | Differentiator | What makes you unique |
| 4-5 | Secondary features | Additional value |
| Last | Social proof | Reviews, awards |

**Design Tips:**
- [ ] Use captions/callouts (40% of users don't read descriptions)
- [ ] Show the app in context (realistic usage)
- [ ] Consistent visual style across all
- [ ] Portrait orientation for phones
- [ ] A/B test different styles

### Preview Video

**Apple App Store:**
- 15-30 seconds maximum
- No text overlays in first few seconds (autoplay muted)
- Show real app usage
- Loop-friendly ending

**Google Play:**
- YouTube video link
- Can be longer form
- Promo video appears prominently

**Video Best Practices:**
- Hook viewers in first 3 seconds
- Show core value proposition
- Include UI and real usage
- Professional quality matters
- Include captions (muted autoplay)

---

## App Description & Metadata

### App Store Description Structure

```
FIRST FOLD (Above the "More" button - 167 chars on iOS):
├── Value proposition
└── Call-to-action or key feature

FULL DESCRIPTION:
├── Key Features (bullet points)
├── Social proof ("1M+ downloads", "Featured by Apple")
├── Primary use cases
├── Secondary features
├── Testimonials or reviews
└── Call-to-action
```

### Description Writing Tips

**Do:**
- ✅ Lead with benefits, not features
- ✅ Use bullet points for scannability
- ✅ Include keywords naturally
- ✅ Update regularly (signals active development)
- ✅ Use emojis sparingly (can increase readability)

**Don't:**
- ❌ Keyword stuff
- ❌ Make false claims
- ❌ Use ALL CAPS excessively
- ❌ Include prices (can change)
- ❌ Mention competitor names negatively

### What's New Section

Update with each release:
- Highlight new features
- Bug fixes (briefly)
- User-requested improvements
- Keep it fresh (signals active app)

---

## Ratings & Reviews

### Rating Strategy

**Target Metrics:**
| Rating | Impact |
|--------|--------|
| 4.5+ | Excellent, conversion boost |
| 4.0-4.4 | Good, competitive |
| 3.5-3.9 | Needs improvement |
| <3.5 | Critical, address urgently |

### Review Solicitation Best Practices

**When to ask for reviews:**
- After positive experience (completed task, achievement)
- After return usage (not first session)
- After sufficient usage time
- Not during friction moments

**System Review Prompts:**
```swift
// iOS - Use SKStoreReviewController (3 prompts/year limit)
SKStoreReviewController.requestReview()

// Android - Use In-App Review API
val manager = ReviewManagerFactory.create(context)
manager.requestReviewFlow()
```

**Custom Prompts (Pre-filter):**
- Ask "Are you enjoying [App]?" first
- If positive → direct to store review
- If negative → direct to feedback form
- Prevents negative reviews, captures feedback

### Responding to Reviews

| Review Type | Response Strategy |
|-------------|-------------------|
| Positive (4-5★) | Thank user, invite sharing |
| Constructive (3★) | Address concerns, ask for details |
| Negative (1-2★) | Apologize, provide support contact |
| Bug reports | Acknowledge, give timeline if possible |

**Response Templates:**

Positive:
> "Thank you for the kind words! We're thrilled you're enjoying the app. If you'd like to share it with friends, we'd greatly appreciate it!"

Negative:
> "We're sorry to hear about your experience. We'd like to help resolve this. Please reach out to support@example.com so we can assist you directly."

---

## Localization

### Priority Markets

| Region | Opportunity | Complexity |
|--------|-------------|------------|
| English (US) | Baseline | - |
| German (DE) | High value | Medium |
| Japanese (JP) | High value | High |
| French (FR) | Medium | Medium |
| Spanish (ES) | High reach | Medium |
| Portuguese (BR) | Growing | Medium |
| Korean (KR) | High value | High |
| Chinese (CN/TW) | Massive but complex | Very High |

### Localization Checklist

For each target market:
- [ ] Translate app name (if appropriate)
- [ ] Translate subtitle/short description
- [ ] Translate full description with local keywords
- [ ] Create localized screenshots with local language
- [ ] Localize preview video (if applicable)
- [ ] Research local keyword variations
- [ ] Consider cultural differences in messaging

### Localization Tips

1. **Don't just translate** - Localize (adapt messaging for culture)
2. **Use native speakers** - Machine translation is obvious
3. **Research local keywords** - Direct translations may not match search
4. **Prioritize by ROI** - Start with high-value, low-complexity markets
5. **Test incrementally** - Launch one market at a time

---

## ASO Tools

| Tool | Purpose | Price Range |
|------|---------|-------------|
| App Annie | Market intelligence | $$ |
| Sensor Tower | Keyword research, rankings | $$ |
| AppFollow | Reviews, keywords | $-$$ |
| Mobile Action | ASO optimization | $ |
| AppTweak | Keyword optimization | $-$$ |
| SplitMetrics | A/B testing | $$ |

---

## Quick Reference Table

| Element | Priority | Update Frequency |
|---------|----------|-----------------|
| App Icon | 🔴 Critical | Rare (major updates) |
| Screenshots | 🔴 Critical | Major updates, seasonally |
| App Name | 🔴 Critical | Rarely (keeps rankings) |
| Keywords | 🟠 High | Monthly optimization |
| Description | 🟠 High | With releases |
| Preview Video | 🟡 Medium | Major updates |
| What's New | 🟡 Medium | Every release |
| Ratings Prompt | 🟡 Medium | Ongoing optimization |
