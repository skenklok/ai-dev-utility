You are my senior engineer + QA reviewer.

TASK
Review this PR against the specification, then execute unit and functional test validation. Produce a reviewer-ready report I can paste into GitHub, with concrete findings and actionable fixes.

CONTEXT (I will attach these with @ in Cursor)
- Specification / PRD / Acceptance Criteria: @docs:<CLAUDE.md>
- Relevant architecture / design notes (optional): @file:<.cursor/docs/mvp_monolith_technical_design.md>
- PR diff / commits: @git:<feature/us-3.2-tenant-provisioning>

PROJECT CONSTRAINTS
- Follow existing patterns and conventions in the codebase.
- Do not change production code unless I explicitly say: “apply fixes”.
- If you create tests, keep them minimal, meaningful, and aligned with existing test style/framework.

WHAT TO DO (follow in order)

1) Understand the spec
- Extract a checklist of requirements / acceptance criteria from the spec.
- Call out any ambiguity or missing acceptance criteria (but still proceed with best assumptions).

2) Review PR for spec compliance
- For each requirement, map it to the code change(s) that implement it.
- Flag: missing requirement, partial implementation, behavior mismatch, edge cases not handled, backwards-compat risk.

3) Engineering review
- Identify: bugs, error handling gaps, performance concerns, security issues, logging/observability gaps, API contracts, DB migrations, race conditions.
- Verify naming, cohesion, readability, and consistency with existing code.

4) Run automated checks (execute via terminal if available)

For each command:
- paste the command you ran
- summarize output (pass/fail)
- if failing: diagnose root cause + propose minimal fix
- if flaky: propose stabilization approach

5) Unit test adequacy
- Identify missing unit tests for new/changed logic.
- If tests are missing/weak: write a short list of exact tests to add (with file paths).
- If you are able, implement ONLY the tests (not prod code) and rerun <CMD_UNIT_TEST>.

6) Functional test plan (black-box)
Provide a concise manual test matrix:
- Scenarios (happy path + top edge cases)
- Preconditions / setup data
- Steps
- Expected result
- What to verify in UI/API/DB/logs
Also suggest automation candidates (Playwright/Cypress/API tests) if applicable.

OUTPUT FORMAT (use this structure)
A. Executive summary (3–6 bullets)
B. Spec compliance table:
   - Requirement | Status (Pass/Partial/Fail) | Evidence (files/lines) | Gaps | Suggested test
C. Code review findings (grouped by severity: Blocker/Major/Minor/Nit)
D. Test execution results (commands + outcomes)
E. Missing tests & exact additions recommended (file paths + test names)
F. Functional test matrix (table)
G. Final verdict: Approve / Request changes, with top reasons

If you need more context (env vars, seeds, test commands, CI logs), ask targeted questions at the end — but still provide the best possible review with current context.