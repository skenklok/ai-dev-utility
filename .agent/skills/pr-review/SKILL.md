---
name: pr-review
description: |
  Perform comprehensive Pull Request reviews against specifications. Use this skill when:
  - User asks to review a PR, branch, or code changes
  - User mentions "PR review", "code review", or "review this branch"
  - User wants to validate changes against a specification or PRD
  - User needs test validation, security review, or compliance checking
  
  This skill covers: spec compliance, engineering review, code complexity analysis,
  automated test execution, unit test adequacy, functional test planning, security review,
  and producing GitHub-ready review reports.
---

# PR Review Skill

## Your Role

You are a senior engineer + QA reviewer. Your task is to review PRs against specifications, execute test validation, and produce GitHub-ready reports with concrete findings and actionable fixes.

---

## STEP 1: Gather Context

**DO THIS FIRST before any review:**

```
REQUIRED CONTEXT:
1. Specification: Look for CLAUDE.md, prd.json, or ask user
2. Branch/PR: Get the branch name from user or detect current branch
3. Design docs: Check .cursor/docs/ or docs/ for architecture
4. Test commands: Check package.json scripts or Makefile
```

**If context is missing, ASK:**
```
I need the following to perform a thorough review:
1. Which branch/PR should I review?
2. Where is the specification or PRD?
3. Are there design docs I should reference?
```

---

## STEP 2: Extract Requirements

**ACTION:** Read the spec and create a requirements checklist.

```bash
# First, read the spec file
view_file <spec_path>
```

**OUTPUT this table:**

| # | Requirement | Source | Notes |
|---|-------------|--------|-------|
| R1 | [Description] | [file:section] | |
| R2 | [Description] | [file:section] | ⚠️ Ambiguous - assuming X |

**IF spec is unclear:** Note ambiguities but proceed with best assumptions.

---

## STEP 3: Get the PR Diff

**ACTION:** Retrieve all changed files.

```bash
# Get list of changed files
git diff main...<branch> --name-only

# Get full diff for review
git diff main...<branch>

# Or for specific files
git diff main...<branch> -- <file_path>
```

**TRACK:** Count lines added/removed for complexity assessment.

---

## STEP 4: Review for Spec Compliance

**ACTION:** For EACH requirement, find the code that implements it.

**CHECK:**
- ✅ Requirement fully implemented
- ⚠️ Partial implementation (note what's missing)
- ❌ Not implemented
- 🔄 Backwards compatibility risk
- 🔓 Edge case not handled

**OUTPUT this table:**

| Requirement | Status | Evidence (file:lines) | Gaps | Suggested Test |
|-------------|--------|----------------------|------|----------------|
| R1: [Name] | ✅ | `src/foo.ts:45-67` | - | should_do_x |
| R2: [Name] | ⚠️ | `src/bar.ts:12` | Missing null check | when_input_null |

---

## STEP 5: Engineering Review

**ACTION:** Review each changed file for issues.

**USE THIS CHECKLIST:**

```
FOR EACH FILE, CHECK:
[ ] Logic errors (off-by-one, null handling, type coercion)
[ ] Error handling (try/catch, promise rejection, error messages)
[ ] Performance (N+1 queries, unnecessary loops, memory leaks)
[ ] Security (injection, auth, data exposure) → Load references/security-checklist.md
[ ] Naming (clear, consistent with codebase)
[ ] Patterns (matches existing code style)
```

**COMPLEXITY ANALYSIS:**

| Indicator | Threshold | Flag If Exceeded |
|-----------|-----------|------------------|
| Function length | > 50 lines | 🔴 Extract methods |
| Nesting depth | > 3 levels | 🔴 Flatten with early returns |
| Cyclomatic complexity | > 10 branches | 🔴 Simplify logic |
| Parameters | > 4 | 🟡 Use options object |

**ALWAYS cite specific file:line references for every finding.**

---

## STEP 6: Run Automated Checks

**ACTION:** Execute these commands and report results.

```bash
# Detect project type and run appropriate commands

# JavaScript/TypeScript
npm run lint
npm run type-check  # or: npx tsc --noEmit
npm run test

# Python
ruff check .
pytest

# Ruby
bundle exec rubocop
bundle exec rspec
```

**FOR EACH COMMAND, REPORT:**

| Check | Command | Result | Notes |
|-------|---------|--------|-------|
| Lint | `npm run lint` | ✅/❌ | [error summary if failed] |
| Types | `npm run type-check` | ✅/❌ | [error location if failed] |
| Tests | `npm run test` | ✅/❌ | [X/Y passing] |

**IF FAILING:** Diagnose root cause and propose minimal fix.

---

## STEP 7: Assess Test Adequacy

**ACTION:** Check if new/changed code has tests.

```bash
# Find test files related to changed files
find . -name "*.test.*" -o -name "*.spec.*" | grep <component_name>

# Read existing tests
view_file <test_file_path>
```

**EVALUATE:**
- [ ] New logic has corresponding tests
- [ ] Tests cover happy path
- [ ] Tests cover edge cases (null, empty, boundary)
- [ ] Tests cover error paths
- [ ] Tests are meaningful (not just for coverage)

**OUTPUT missing tests:**

| File | Missing Test | Priority |
|------|--------------|----------|
| `src/auth.ts` | `should_reject_expired_token` | 🔴 High |
| `src/utils.ts` | `should_handle_null` | 🟡 Medium |

**IF POSSIBLE:** Write the missing tests (not prod code).

---

## STEP 8: Create Functional Test Plan

**ACTION:** Create a manual test matrix for QA.

**OUTPUT:**

| # | Scenario | Preconditions | Steps | Expected | Verify |
|---|----------|---------------|-------|----------|--------|
| 1 | Happy path | User logged in | 1. Do X 2. Click Y | Success | DB: record exists |
| 2 | Empty input | - | 1. Submit empty form | Error shown | UI: validation message |
| 3 | [Edge case] | [Setup] | [Steps] | [Result] | [Where to check] |

---

## STEP 9: Security Review

**ACTION:** Check for security vulnerabilities.

```bash
# Load the security checklist
view_file references/security-checklist.md
```

**RED TEAM - CHECK EACH:**
- [ ] Auth bypass possible?
- [ ] IDOR (can access other users' data)?
- [ ] SQL/XSS/Command injection?
- [ ] Sensitive data in logs/responses?
- [ ] Rate limiting on sensitive endpoints?

---

## STEP 10: Generate Final Report

**ACTION:** Compile findings into GitHub-ready format.

**USE THIS EXACT STRUCTURE:**

```markdown
# PR Review: [Branch Name]

## A. Executive Summary
- 🟢/🟡/🔴 [3-6 bullet findings]
- **Verdict:** Approve / Request Changes

## B. Spec Compliance

| Requirement | Status | Evidence | Gaps |
|-------------|--------|----------|------|
| ... | ✅/⚠️/❌ | file:line | ... |

## C. Complexity

| Metric | Value |
|--------|-------|
| LOC Added | ~X |
| LOC Removed | ~X |
| Rating | Low/Medium/High |

### Hotspots
| Location | Issue | Fix |
|----------|-------|-----|

## D. Findings (by severity)

### 🔴 Blockers
1. **[Title]** - `file:line`
   - Issue: ...
   - Fix: ...

### 🟠 Major
### 🟡 Minor
### 💭 Nits

## E. Automated Checks

| Check | Command | Result |
|-------|---------|--------|

## F. Missing Tests

| File | Test | Priority |
|------|------|----------|

## G. Manual Test Matrix

| # | Scenario | Steps | Expected |
|---|----------|-------|----------|

## H. Verdict

**Decision:** ✅ Approve / ❌ Request Changes

**Reasons:**
1. ...
2. ...
```

---

## CRITICAL RULES

```
⚠️ ALWAYS:
- Cite file:line for every finding
- Run commands, don't just suggest them
- Verify findings against actual code
- Use evidence, not assumptions
- Be specific and actionable

🚫 NEVER:
- Change production code without explicit permission
- Auto-apply fixes
- Make vague criticisms
- Over-engineer suggestions
- Ignore existing codebase patterns
```

---

## SEVERITY LABELS

Use these consistently:
- 🔴 **Blocker** - Must fix before merge
- 🟠 **Major** - Should fix, significant issue
- 🟡 **Minor** - Nice to fix, not blocking
- 💭 **Nit** - Optional, stylistic
- 💡 **Suggestion** - Idea for future
- ❓ **Question** - Need clarification

---

## REFERENCES

Load these when needed:
- `references/security-checklist.md` - Security vulnerabilities
- `references/test-patterns.md` - Test coverage patterns
- `references/complexity-guide.md` - Complexity thresholds
