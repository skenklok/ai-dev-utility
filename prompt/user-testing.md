You are my QA lead + senior engineer. Your job is to test this product “as a user” based on the specification, not just run unit tests.

SPECS & RULES (attach these as context)
- Project rules/specs: @Folders .cursor/
- Agent guidance: @Files CLAUDE.md (and/or the CLAUDE.md folder)
- Implementation spec: @Folders docs/

EXECUTION CAPABILITY
Use Cursor Agent terminal to run commands and execute tests. If terminal access is restricted, still produce the full test plan + exact commands I should run. (Cursor Agent can run terminal commands when enabled.) 

GOAL
Perform a spec-driven regression test:
1) prove key user journeys still work end-to-end
2) prove critical acceptance criteria still hold
3) identify regressions with repro steps + fixes

PROCESS (do in order)

1) Build “User Journey Checklist” FROM THE SPEC
- Read .cursor/, CLAUDE.md, and docs/
- Extract the top user journeys (happy path) + critical edge cases.
- For each journey, write concrete acceptance checks (“Given/When/Then” style is fine).
- This checklist is the source of truth for the regression.

2) Discover how to run the app & existing test tooling
- Inspect repo tooling (package.json/scripts, Makefile, README, CI config).
- Determine: install, lint, build/typecheck, unit tests, and any E2E framework already present (Playwright/Cypress/etc).

3) Run automated regression in layers (fast → realistic)
A) Install + lint/format + build/typecheck
B) Unit tests
C) “As user” tests:
   - If E2E tests already exist: run them and map coverage to the checklist.
   - If NO E2E suite exists: scaffold the smallest E2E harness using the repo’s stack (prefer Playwright or Cypress) and implement tests for ONLY the top critical user journeys from the checklist.
   - Keep selectors stable (prefer data-test/data-testid/data-cy if present). Favor testing workflows over implementation details.

4) Execute the user journeys
- Start the app locally in a test mode (seed data if needed).
- Run E2E flows headless (and headed only when debugging).
- For each failed journey: capture exact steps, expected vs actual, and the likely root cause (files/lines).

5) Output a spec-mapped report (copy/paste friendly)
A. User Journey Checklist (from spec)
B. Test runs (exact commands) + outcomes
C. Coverage map: Journey → E2E test file(s) / assertions
D. Regressions found (Blocker/Major/Minor) with repro steps
E. Suggested minimal fixes (DO NOT change prod code unless I say “apply fixes”)
F. Gaps: what is still untested + recommended next tests
G. Final verdict: USER-REGRESSION PASS / FAIL

CONSTRAINTS
- Do not modify production code unless I explicitly say: “apply fixes”.
- You may add tests if needed, minimal and consistent with repo patterns.