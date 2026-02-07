# Test Adequacy Patterns

Guide for evaluating and recommending tests during PR reviews.

## Test Coverage Evaluation

### What Should Be Tested

| Code Type | Required Tests | Priority |
|-----------|---------------|----------|
| Business logic | Unit tests | 🔴 High |
| API endpoints | Integration tests | 🔴 High |
| Data transformations | Unit tests | 🔴 High |
| Edge cases | Unit tests | 🟠 Medium |
| Error handling | Unit + Integration | 🟠 Medium |
| UI components | Component tests | 🟡 Low-Medium |
| Utilities/helpers | Unit tests | 🟡 Medium |

### Test Quality Indicators

**Good Tests:**
- ✅ Test one thing (single assertion focus)
- ✅ Descriptive names (`should_reject_when_token_expired`)
- ✅ Arrange-Act-Assert pattern
- ✅ Independent (no order dependency)
- ✅ Fast execution

**Weak Tests:**
- ❌ Multiple unrelated assertions
- ❌ Vague names (`test1`, `it works`)
- ❌ No edge cases
- ❌ Only happy path
- ❌ Mocking everything (no integration value)

## Common Missing Test Patterns

### 1. Input Validation
```typescript
// Missing tests for:
describe('validateInput', () => {
  it('should reject empty input');
  it('should reject null/undefined');
  it('should reject invalid format');
  it('should accept valid input');
});
```

### 2. Error Handling
```typescript
// Missing tests for:
describe('fetchData', () => {
  it('should handle network errors');
  it('should handle timeout');
  it('should handle malformed response');
  it('should handle 404/500 status codes');
});
```

### 3. Boundary Conditions
```typescript
// Missing tests for:
describe('pagination', () => {
  it('should handle empty results');
  it('should handle single item');
  it('should handle exactly page size');
  it('should handle last page partial');
});
```

### 4. State Transitions
```typescript
// Missing tests for:
describe('orderStatus', () => {
  it('should transition pending -> processing');
  it('should reject invalid transitions');
  it('should handle concurrent updates');
});
```

### 5. Authentication/Authorization
```typescript
// Missing tests for:
describe('protectedRoute', () => {
  it('should reject unauthenticated requests');
  it('should reject expired tokens');
  it('should reject insufficient permissions');
  it('should allow authorized users');
});
```

## Test Recommendation Format

When recommending missing tests:

```markdown
### Missing Tests

| File | Test Name | Description | Priority |
|------|-----------|-------------|----------|
| `src/services/__tests__/auth.test.ts` | `should_reject_expired_token` | Verify expired JWT rejected | High |
| `src/utils/__tests__/format.test.ts` | `should_handle_null_input` | Edge case for formatter | Medium |
```

## Test File Naming Conventions

| Framework | Convention | Example |
|-----------|-----------|---------|
| Jest | `*.test.ts` or `__tests__/` | `auth.test.ts` |
| Vitest | `*.test.ts` or `*.spec.ts` | `auth.spec.ts` |
| RSpec | `*_spec.rb` in `spec/` | `auth_spec.rb` |
| Pytest | `test_*.py` or `*_test.py` | `test_auth.py` |
| Playwright | `*.spec.ts` in `e2e/` | `login.spec.ts` |

## Quick Checklist

For each new/changed function:
- [ ] Happy path tested
- [ ] Error cases tested
- [ ] Edge cases tested (null, empty, boundary)
- [ ] Assertions are meaningful
- [ ] Test is independent and repeatable
