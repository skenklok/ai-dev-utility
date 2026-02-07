---
name: implementation-review-before-commit
description: |
  Reviews staged implementation for simplicity, low complexity, good patterns, and requirement
  alignment before committing. Triggers on code review requests, implementation review,
  pre-commit checks, complexity analysis, or quality review.
---

# Implementation review before commit (simplicity + requirements)

## Scope

- Staged diff is the source of truth.
- Optimize for: clarity, low cyclomatic complexity, maintainability, and meeting requirements.

## Default workflow

1. Collect staged diff + changed file list
   - `git diff --cached`
   - `git diff --cached --name-only`

2. Reconstruct intent
   - What requirement is being implemented or fixed?
   - What are the acceptance criteria implied by the diff?
   - State assumptions if requirements are not present.

3. Review for simplicity and patterns
   - Use the checklist in `reference/implementation-checklist.md` (load if needed).
   - Specifically check:
     - unnecessary abstraction, premature generalization
     - deeply nested conditionals (suggest guard clauses)
     - duplicated logic (extract small helpers)
     - error handling consistency
     - boundary validation (inputs, nulls, types)
     - performance footguns in hot paths
     - readability: naming, file organization, “one responsibility” per function

4. Requirement trace
   - For each inferred requirement, point to the code that fulfills it.
   - Call out gaps or “looks implemented but not actually enforced”.

5. Report
   - PASS / WARN / FAIL
   - “Meets requirements?” yes/no with evidence
   - Top 3 complexity reductions (smallest diffs)
   - Any risky patterns with safer alternatives

## Guardrails

- Default to minimal-change suggestions (small PR/commit friendliness).
- Don’t bikeshed style; focus on correctness, simplicity, and requirements.
- Don’t claim compliance with a requirement unless the diff shows it.
