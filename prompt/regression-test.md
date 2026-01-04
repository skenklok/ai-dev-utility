You are my QA + senior engineer.

GOAL
Run a regression test pass to ensure the codebase still satisfies the specification and hasn’t broken existing functionality. Produce a clear report of what you ran, what passed/failed, and what needs fixing.

CONTEXT (attach these via Cursor @Files & Folders before you start)
- All project rules/specs: @Files & Folders -> .cursor/
- Agent / project guidance: @Files & Folders -> CLAUDE.md (or the CLAUDE.md folder if that’s how it’s stored)
- Implementation specification: @Files & Folders -> docs/

IMPORTANT
- Follow any repo rules in .cursor/ as the source of truth for how to run tests and how to format output. (Cursor supports project rules and attaching files/folders into context via @ mentions.)  [oai_citation:0‡Cursor](https://cursor.com/docs/context/rules?utm_source=chatgpt.com)
- Do NOT change production code unless I explicitly say: “apply fixes”.
- If tests are missing, you may add tests (preferred) but keep changes minimal and consistent with existing patterns.

WHAT I WANT YOU TO DO (in order)

1) Build a regression checklist
- Read specs from .cursor/ + CLAUDE.md + docs/ and extract a “Regression Coverage Checklist”:
  - Core user flows / key behaviors
  - Critical edge cases
  - Any explicit non-regression requirements
- If something is ambiguous, state the assumption and continue.

2) Discover how this repo runs tests
- Inspect package.json / pyproject / Makefile / scripts / README / CI config to determine:
  - Install command
  - Lint/format command (if any)
  - Build/typecheck command (if any)
  - Unit test command
  - Integration/E2E/functional test command (if any)
- If multiple exist, choose the one used by CI or most standard for the repo and explain why.

3) Execute automated regression (run commands)
Run in this sequence. If a step doesn’t exist, skip it and explain:
A. Install dependencies
B. Lint / formatting checks
C. Typecheck / build
D. Unit tests
E. Integration/E2E/functional tests (Playwright/Cypress/etc. if present)

For each command:
- Show the exact command you ran
- Summarize result (pass/fail)
- If fail: root cause analysis + smallest fix recommendation
- If flaky: stabilization suggestion

4) Targeted spec-based regression
- Based on the checklist (Step 1) and the PR/changes currently in the working tree:
  - Identify the most likely regression areas
  - Ensure tests cover them, or propose/implement missing tests (tests-only changes allowed)

5) Manual functional regression plan (only if needed)
Provide a concise manual test matrix for the key flows:
- Preconditions/setup data
- Steps
- Expected results
- What to verify (UI/API/DB/logs)

OUTPUT FORMAT (exact)
A. Regression Coverage Checklist (from spec)
B. Commands executed + outcomes (copy/paste friendly)
C. Failures / risks (Blocker/Major/Minor)
D. Missing test gaps (with file paths + test names to add)
E. Manual functional regression matrix (if applicable)
F. Final verdict: “Regression PASS” or “Regression FAIL” + top reasons

If you need env vars, test accounts, seeds, or services running to complete the regression, ask at the end — but still complete as much as possible now.