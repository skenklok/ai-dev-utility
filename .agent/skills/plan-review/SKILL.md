---
name: plan-review
description: |
  Review implementation plans, code changes, or architectural decisions before making changes. Use this skill when:
  - User asks to review a plan, design, or proposed changes
  - User mentions "plan mode", "review before implementing", or "evaluate this approach"
  - User wants detailed analysis of tradeoffs before committing to a direction
  - User needs architecture, code quality, test, or performance review
  
  This skill covers: architecture review, code quality review, test review, 
  performance review, issue analysis with options, and interactive feedback workflow.
---

# Plan Review Skill

Review plans thoroughly before making any code changes. For every issue or recommendation, explain the concrete tradeoffs, give an opinionated recommendation, and ask for input before assuming a direction.

## Engineering Preferences

Use these to guide your recommendations:

- **DRY is important** — Flag repetition aggressively
- **Well-tested code is non-negotiable** — I'd rather have too many tests than too few
- **"Engineered enough"** — Not under-engineered (fragile, hacky) and not over-engineered (premature abstraction, unnecessary complexity)
- **Handle edge cases** — Err on the side of handling more edge cases, not fewer
- **Thoughtfulness > speed** — Take time to consider implications
- **Explicit over clever** — Bias toward readable, obvious code

---

## Review Sections

### 1. Architecture Review

Evaluate:
- [ ] Overall system design and component boundaries
- [ ] Dependency graph and coupling concerns
- [ ] Data flow patterns and potential bottlenecks
- [ ] Scaling characteristics and single points of failure
- [ ] Security architecture (auth, data access, API boundaries)

### 2. Code Quality Review

Evaluate:
- [ ] Code organization and module structure
- [ ] DRY violations — be aggressive here
- [ ] Error handling patterns and missing edge cases (call these out explicitly)
- [ ] Technical debt hotspots
- [ ] Areas that are over-engineered or under-engineered relative to preferences

### 3. Test Review

Evaluate:
- [ ] Test coverage gaps (unit, integration, e2e)
- [ ] Test quality and assertion strength
- [ ] Missing edge case coverage — be thorough
- [ ] Untested failure modes and error paths

### 4. Performance Review

Evaluate:
- [ ] N+1 queries and database access patterns
- [ ] Memory-usage concerns
- [ ] Caching opportunities
- [ ] Slow or high-complexity code paths

---

## Issue Handling

For every specific issue (bug, smell, design concern, or risk):

1. **Describe the problem concretely**, with file and line references
2. **Present 2-3 options**, including "do nothing" where that's reasonable
3. **For each option, specify:**
   - Implementation effort
   - Risk
   - Impact on other code
   - Maintenance burden
4. **Give your recommended option and why**, mapped to preferences above
5. **Explicitly ask** whether user agrees or wants a different direction before proceeding

---

## Workflow and Interaction

### Key Rules

- **Do not assume priorities on timeline or scale**
- **After each section, pause and ask for feedback before moving on**

### Before Starting

Ask if user wants one of two options:

**1/ BIG CHANGE Mode:**
Work through interactively, one section at a time:
```
Architecture → Code Quality → Tests → Performance
```
With at most 4 top issues in each section.

**2/ SMALL CHANGE Mode:**
Work through interactively ONE question per review section.

### For Each Stage of Review

1. Output the explanation and pros/cons of each stage's questions
2. Give your opinionated recommendation and why
3. Use structured questions with numbered issues and lettered options:

```
**Issue #1: [Issue Title]**
[Description with file:line references]

Options:
A) [Recommended] [Option description]
   - Effort: Low | Medium | High
   - Risk: Low | Medium | High
   - Why recommended: [Explanation]

B) [Alternative option]
   - Effort: ...
   - Risk: ...
   - Trade-off: [Explanation]

C) Do nothing
   - Risk: [What happens if ignored]

👉 My recommendation: Option A because [reason mapped to preferences]

Do you agree, or would you prefer a different direction?
```

> **Note:** Make the recommended option always the 1st option (A). Clearly label issue NUMBER and option LETTER so the user doesn't get confused.

---

## Output Format

### Section Header
```markdown
## [Section Number]. [Section Name] Review

### Summary
[Brief overview of findings for this section]

### Issues Found

**Issue #1: [Title]**
...

**Issue #2: [Title]**
...

---

**Section Complete.** Ready for your feedback before proceeding to the next section.
What would you like me to do?
- [ ] Proceed to next section
- [ ] Dive deeper into any issue above
- [ ] Revise recommendations
```

---

## Quick Reference

| Section | Focus Areas | Max Issues (BIG CHANGE) |
|---------|-------------|------------------------|
| Architecture | Boundaries, coupling, data flow, security | 4 |
| Code Quality | DRY, errors, debt, complexity | 4 |
| Tests | Coverage, quality, edge cases | 4 |
| Performance | N+1, memory, caching, complexity | 4 |
