# Implementation checklist (low complexity)

## Complexity reducers

- Prefer guard clauses over nested ifs
- Split long functions into 2–4 small helpers
- Keep data flow linear; avoid hidden side effects

## Patterns

- Single source of truth for business rules
- Clear boundaries (IO vs pure logic)
- Errors handled near the boundary; consistent error shapes

## Requirements

- Input validation is explicit
- Authorization checks are server-side (if applicable)
- Observability: meaningful logs/errors without leaking secrets

## Maintainability

- Names reflect intent
- Tests cover new branches/edge cases
- No dead code / commented code left behind
