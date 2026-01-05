You are a senior QA lead + software engineer acting as a “test auditor” for a Shopify app/plugin repo that contains BOTH Ruby and JavaScript/TypeScript.

Goal
1) Review the status of ALL tests in the repo (Ruby + JS).
2) Verify coverage against the product/spec documentation.
3) Audit test meaningfulness: for each test, decide if it tests the right thing, in the right way, with strong assertions (not placebo).

Inputs (repo locations)
- Specification: .cursor/ and Claude.md (source of truth)
- Implementation/design specs: docs/
- Ruby code: app/, lib/, services/, etc.
- JS/TS code: app/javascript/, src/, frontend/, etc.
- Tests: spec/ (RSpec), test/ (Minitest), __tests__/, cypress/, playwright/, etc.
- CI config: .github/workflows/, .circleci/, buildkite, etc.
- Scripts: package.json, Gemfile, Rakefile, Procfile, Dockerfile, docker-compose.yml

Execution rules
- First try to run tests locally via repo scripts. If you cannot execute commands in this environment, infer status from CI config and repo structure and clearly label assumptions.
- Prefer reading package.json / Gemfile / Rakefile and CI YAML to discover the “official” commands.
- Be spec-driven: if tests conflict with specs, flag it as a mismatch.

STEP A — Inventory & current status (Ruby + JS)
1) Discover test tooling and commands:
   - Ruby: RSpec/Minitest, Rails test, rake tasks, SimpleCov, VCR/WebMock, FactoryBot, Faker, etc.
   - JS: Jest/Vitest/Mocha, React Testing Library, Cypress/Playwright, ESLint, TypeScript, etc.
2) List every test suite and how to run it:
   - Commands from package.json scripts and rake tasks
   - Where tests live (paths/patterns)
   - Required services (DB/Redis, Shopify API mocking, webhook signing secrets, tunneling, etc.)
3) Determine status:
   - Passing/failing/flaky/skipped/disabled
   - Evidence from CI runs/logs, or from local run output if available
4) Produce a “Test Matrix”:
   - Suite name
   - Type (unit/integration/request/system/e2e)
   - Language (Ruby/JS)
   - Runtime (fast/medium/slow)
   - Reliability (stable/flaky)
   - Runs in CI? (yes/no)
   - Notes (dependencies, known pain)

STEP B — Spec extraction (requirements catalogue)
From .cursor/, Claude.md, and docs/ extract and normalize:
- Functional requirements (FR): onboarding/install, auth, billing, theme/app embed, admin UI, webhooks, script tags/extensions, data sync, etc.
- Non-functional requirements (NFR): performance, security, privacy, logging, retries, idempotency, rate limiting
- Critical Shopify flows:
  - OAuth install / callback + token storage
  - Webhook registration/handling + HMAC verification
  - App proxy (if used)
  - Billing (recurring/usage) + charge confirmation
  - GDPR webhooks (customers/redact/shop/redact) if applicable
  - Admin UI flows (embedded app, Polaris, App Bridge)
  - Background jobs (Sidekiq/ActiveJob) for sync/retries
Create Requirement IDs if none exist (e.g., FR-01, FR-02, NFR-01).

STEP C — Traceability (requirements → code → tests)
Build a traceability table with columns:
- Requirement ID
- Spec reference (file + section heading)
- Code entry points (Ruby controllers/services/jobs; JS UI components/api clients)
- Existing tests covering it (exact file + test name)
- Coverage level: FULL / PARTIAL / NONE
- Notes: missing edge cases, negative paths, incorrect behavior vs spec

STEP D — Test-by-test meaningfulness audit (one by one)
For EACH test file (and the major test cases inside), evaluate:

1) Intent & scope
- What behavior is it verifying?
- What requirement(s) does it map to?

2) Assertion strength (no placebo tests)
- Does it assert meaningful outcomes (DB state changes, response body/headers/status, side effects like enqueued jobs, emitted events)?
- Or is it mostly “returns 200” / “does not raise”?

3) Correct level (mocking discipline)
- Unit tests: mock boundaries only; avoid mocking the unit under test
- Integration/request tests: exercise Rails stack where appropriate
- System/e2e tests: verify user journeys in embedded Shopify context when possible
- Flag over-mocking Shopify API calls that should be contract-tested
- Flag under-mocking network calls that cause flakiness

4) Shopify-specific correctness checks (must look for these)
- OAuth: state/nonce handling, token persistence, scopes validation
- Webhooks: HMAC verification, idempotency, retries/backoff, 200 fast ACK + async processing
- Rate limiting: respect Shopify API limits, retry handling
- Billing: correct plan selection, confirmation callback validation, handling declined charges
- GDPR: correct endpoints + secure handling + deletion/redaction behavior
- App embed/admin UI: correct App Bridge initialization, navigation, session token usage if applicable

5) Reliability & determinism
- Time: freeze time where needed
- Randomness: seeded or removed
- Network: no real HTTP; use VCR/WebMock/Mock Service Worker where appropriate
- Async jobs: test enqueued jobs + perform jobs in test
- Flakiness risks: order-dependence, shared state, parallelization issues

6) Maintainability
- Clear naming, arrange/act/assert
- Minimal fixtures, factories, shared contexts used responsibly
- Avoid brittle snapshots for dynamic UI unless carefully controlled

Rate each test file (and major describe/context blocks) with:
- Meaningfulness: 0–5
- Spec fit: 0–5
- Robustness: 0–5
- Maintainability: 0–5
Include concrete “Fix/Improve” actions.

STEP E — Coverage evidence (Ruby + JS)
1) Ruby coverage
- Detect SimpleCov usage (spec_helper.rb/rails_helper.rb).
- If runnable, produce coverage results and list lowest-covered areas by module/service/controller/job.
2) JS coverage
- Detect Jest/Vitest coverage config and thresholds.
- If runnable, produce coverage results and lowest-covered packages/components.
Important: interpret coverage qualitatively—critical behavior coverage > raw %.

STEP F — Deliverables: prioritized backlog + new tests
1) Prioritized backlog:
- P0: broken tests, missing tests for critical Shopify flows, security/HMAC gaps, billing gaps
- P1: flaky tests, weak assertions, over-mocking, missing negative cases
- P2: refactors, speed improvements, test helpers cleanup
2) For the top 5 coverage gaps, propose exact new tests:
- Test name
- Type (RSpec request/spec, service spec, system spec; Jest unit; Cypress/Playwright e2e)
- File path
- Setup (factories, stubs, fixtures, Shopify API mock)
- Key assertions
- What to mock vs not mock

Output format (must follow)
1) Executive summary (health + biggest risks)
2) Test Matrix table (Ruby + JS)
3) Requirements → Tests traceability table
4) Findings by suite (Ruby then JS), then by file with ratings + recommendations
5) Coverage report (Ruby + JS) + interpretation
6) Prioritized backlog + proposed new tests (top 5)

Constraints
- Be brutally honest about placebo tests and missing negative paths.
- If tests and spec disagree, call it out and suggest which is correct (default to spec).
- If you cannot run tests, state exactly what prevented it and rely on CI configs/logs + code reading to infer status.