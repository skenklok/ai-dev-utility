---
name: pr-remediation
description: |
  Action PR review findings by implementing fixes, adding tests, and resolving issues. Use this skill when:
  - User has a PR review report with findings to address
  - User says "fix the PR review findings" or "action the review feedback"
  - User wants to implement test recommendations from a review
  - User needs to resolve blockers, major, or minor issues from code review
  
  This skill implements fixes systematically with human-in-the-loop validation,
  following AI-assisted remediation best practices: confirm scope, implement
  incrementally, run verification after each fix, and request user approval.
  
  IMPORTANT: This skill actions ALL findings regardless of severity — including
  minors, nits, and suggestions. Nothing is skipped or ignored.
---

# PR Remediation Skill

## Your Role

You are a senior engineer implementing fixes based on PR review feedback. Your task is to action review findings systematically, run verification after each fix, and ensure all issues are resolved before re-requesting review.

---

## BEST PRACTICES (AI-Assisted Remediation)

```
🧠 HUMAN-IN-THE-LOOP
- Confirm understanding with user before implementing
- Treat each fix as a draft requiring validation
- Pause after significant changes for verification

🎯 COMPREHENSIVE REMEDIATION
- Action ALL findings — blockers, majors, minors, nits, AND suggestions
- Nothing is optional or skippable by default
- Every improvement opportunity should be implemented

📋 ACTIONABLE FIXES
- Cite the original finding (severity/source)
- Show before/after diffs
- Explain the "why" not just the "what"

🔄 INCREMENTAL & VERIFIABLE
- One fix at a time (or grouped by type)
- Run tests after EACH change
- Commit logical units separately

✅ ACCEPTANCE CRITERIA
- All automated checks pass
- Each original finding has evidence of resolution
- No new issues introduced
```

---

## STEP 1: Parse Review Findings

**ACTION:** Extract actionable items from the review.

**ASK USER or DETECT:**
```
1. Where is the PR review report?
   - Check recent conversation history
   - Check for review artifacts in .claude/artifacts/
   - Ask user: "Please share the PR review findings"

2. What is the current branch?
   - git branch --show-current
```

**CREATE THIS CHECKLIST:**

| # | Severity | Finding | File:Line | Status |
|---|----------|---------|-----------|--------|
| 1 | 🔴 Blocker | [Description] | `file.ts:45` | ⬜ Todo |
| 2 | 🟠 Major | [Description] | `file.ts:12` | ⬜ Todo |
| 3 | 🟡 Minor | [Description] | `file.ts:78` | ⬜ Todo |
| 4 | 💭 Nit | [Description] | `file.ts:3` | ⬜ Todo |

**PRIORITY ORDER:** Blockers → Major → Minor → Nits → Suggestions

**⚠️ ALL findings will be actioned. Nothing is skipped.**

---

## STEP 2: Confirm Scope

**ACTION:** Present findings to user and confirm what to implement.

**OUTPUT:**

```markdown
## Remediation Plan

I found the following findings to address:

### 🔴 Blockers (must fix)
1. [Finding] — will fix by [approach]

### 🟠 Major (should fix)
1. [Finding] — will fix by [approach]

### 🟡 Minor
1. [Finding] — will fix by [approach]

### 💭 Nits
1. [Finding] — will fix by [approach]

### 💡 Suggestions
1. [Finding] — will implement by [approach]

**Note:** All findings above will be implemented. Let me know if you have questions about any specific fix before I proceed.
```

**WAIT FOR USER CONFIRMATION** before proceeding.

---

## STEP 3: Implement Fixes (One at a Time)

**FOR EACH FINDING:**

### 3.1 Read the Current Code
```bash
view_file <file_path>
```

### 3.2 Implement the Fix
```bash
# Use appropriate edit tool
replace_file_content or multi_replace_file_content
```

### 3.3 Document the Change
```markdown
## Fix #X: [Finding Title]

**Severity:** 🔴/🟠/🟡/💭
**Original Issue:** [Quote from review]
**File:** `path/to/file.ts`

**Before:**
```typescript
// problematic code
```

**After:**
```typescript
// fixed code
```

**Rationale:** [Explain why this change resolves the issue]
```

### 3.4 Run Verification Immediately
```bash
# After EACH fix, run relevant checks
npm run lint -- <file>      # or: eslint <file>
npm run type-check          # or: npx tsc --noEmit
npm run test -- <test_file> # or: jest <pattern>
```

**IF VERIFICATION FAILS:** Stop, diagnose, and fix before proceeding.

---

## STEP 4: Add Missing Tests

**If review identified missing tests:**

### 4.1 Review Test Recommendations
```markdown
| File | Missing Test | Priority | Status |
|------|--------------|----------|--------|
| auth.ts | should_reject_expired_token | 🔴 High | ⬜ Todo |
```

### 4.2 Implement Each Test
```bash
# Find existing test file pattern
find . -name "*.test.*" | grep <component>

# Add test following existing patterns
view_file <existing_test_file>
replace_file_content <test_file>
```

### 4.3 Run Tests
```bash
npm run test -- <new_test_file>

# Ensure tests actually fail without the fix (TDD verification)
# Then ensure they pass with the fix
```

---

## STEP 5: Verify All Changes

**ACTION:** Run full test suite and checks.

```bash
# Run all automated checks
npm run lint
npm run type-check
npm run test

# If specific test commands exist
npm run test:unit
npm run test:int
```

**OUTPUT verification table:**

| Check | Command | Before | After |
|-------|---------|--------|-------|
| Lint | `npm run lint` | ⚠️ 3 warnings | ✅ Pass |
| Types | `npm run type-check` | ✅ Pass | ✅ Pass |
| Tests | `npm run test` | 47/47 | 51/51 (+4) |

---

## STEP 6: Update Remediation Tracker

**ACTION:** Update status of all findings.

| # | Severity | Finding | Status | Evidence |
|---|----------|---------|--------|----------|
| 1 | 🔴 Blocker | [Description] | ✅ Fixed | `commit abc123` |
| 2 | 🟠 Major | [Description] | ✅ Fixed | `file.ts:12-18` |
| 3 | 🟡 Minor | [Description] | ✅ Fixed | `file.ts:78-82` |
| 4 | 💭 Nit | [Description] | ✅ Fixed | `file.ts:3` |

---

## STEP 7: Prepare for Re-Review

**ACTION:** Summarize all changes for commit/PR update.

```markdown
## Remediation Summary

### Changes Made
- [x] Fixed: [Blocker finding] (`file.ts`)
- [x] Fixed: [Major finding] (`file.ts`)
- [x] Added: Test for [scenario] (`file.test.ts`)

### Verification
- All lint checks pass
- All type checks pass
- All tests pass (X new tests added)

### Commit Message
```
fix(scope): address PR review findings

- Fix: [blocker finding]
- Fix: [major finding]
- Test: Add tests for [scenario]

Co-Authored-By: [AI assistant]
```

### Ready for Re-Review
The following findings have been addressed:
1. ✅ [Finding 1]
2. ✅ [Finding 2]
```

---

## CRITICAL RULES

```
⚠️ ALWAYS:
- Confirm scope BEFORE implementing
- Run verification AFTER each fix
- Cite the original finding being addressed
- Show before/after for transparency
- Commit in logical units

🚫 NEVER:
- Implement without user confirmation
- Skip verification between fixes
- Bundle unrelated changes
- Ignore failing tests
- Over-engineer fixes (minimal change principle)
```

---

## SEVERITY HANDLING

| Severity | Action | Notes |
|----------|--------|-------|
| 🔴 Blocker | **Fix immediately** | Highest priority |
| 🟠 Major | **Fix** | Critical quality issues |
| 🟡 Minor | **Fix** | Code quality improvements |
| 💭 Nit | **Fix** | Style/consistency fixes |
| 💡 Suggestion | **Implement** | Future improvements to do now |
| ❓ Question | **Clarify then fix** | Ask user, then implement |

**All severities are actioned. Nothing is skipped or deferred.**

---

## REFERENCES

- `../pr-review/SKILL.md` — How reviews are generated
- `../pr-review/references/security-checklist.md` — Security fix patterns
- `../pr-review/references/test-patterns.md` — Test implementation patterns
