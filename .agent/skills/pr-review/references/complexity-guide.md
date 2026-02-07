# Code Complexity Analysis Guide

Framework for evaluating code complexity during PR reviews.

## Complexity Metrics

### Lines of Code (LOC)

| Change Size | LOC Added | Review Depth |
|-------------|-----------|--------------|
| Small | < 50 | Quick review |
| Medium | 50-200 | Standard review |
| Large | 200-500 | Deep review |
| Very Large | > 500 | Consider splitting PR |

### Cyclomatic Complexity

Measure of independent paths through code:

| Complexity | Rating | Action |
|------------|--------|--------|
| 1-5 | Low | ✅ Good |
| 6-10 | Medium | ⚠️ Review |
| 11-20 | High | 🔴 Refactor |
| > 20 | Very High | 🚫 Must refactor |

**What increases complexity:**
- `if/else` branches (+1 per branch)
- `switch/case` (+1 per case)
- Loops (`for`, `while`, `forEach`) (+1 each)
- Logical operators (`&&`, `||`) (+1 each)
- Ternary operators (+1 each)
- Catch blocks (+1 each)

## Hotspot Indicators

### Function Length
| Lines | Rating | Recommendation |
|-------|--------|----------------|
| < 20 | ✅ Good | - |
| 20-50 | ⚠️ Medium | Consider splitting |
| > 50 | 🔴 Long | Extract methods |

### Nesting Depth
| Levels | Rating | Recommendation |
|--------|--------|----------------|
| 1-2 | ✅ Good | - |
| 3 | ⚠️ Medium | Flatten if possible |
| > 3 | 🔴 Deep | Must flatten |

### Parameter Count
| Count | Rating | Recommendation |
|-------|--------|----------------|
| 0-3 | ✅ Good | - |
| 4-5 | ⚠️ Medium | Consider options object |
| > 5 | 🔴 Many | Use options object |

## Refactoring Strategies

### 1. Extract Method
**Before:**
```typescript
function processOrder(order) {
  // 20 lines of validation
  // 15 lines of calculation
  // 10 lines of formatting
}
```
**After:**
```typescript
function processOrder(order) {
  validateOrder(order);
  const total = calculateTotal(order);
  return formatResult(total);
}
```

### 2. Early Returns (Flatten Nesting)
**Before:**
```typescript
function check(x) {
  if (x) {
    if (x.valid) {
      if (x.active) {
        return process(x);
      }
    }
  }
  return null;
}
```
**After:**
```typescript
function check(x) {
  if (!x) return null;
  if (!x.valid) return null;
  if (!x.active) return null;
  return process(x);
}
```

### 3. Strategy Pattern (Reduce Switch)
**Before:**
```typescript
function calculate(type, value) {
  switch(type) {
    case 'A': return value * 1.1;
    case 'B': return value * 1.2;
    case 'C': return value * 1.3;
    // 10 more cases...
  }
}
```
**After:**
```typescript
const strategies = {
  'A': (v) => v * 1.1,
  'B': (v) => v * 1.2,
  'C': (v) => v * 1.3,
};

function calculate(type, value) {
  return strategies[type]?.(value) ?? value;
}
```

### 4. Options Object (Reduce Parameters)
**Before:**
```typescript
function createUser(name, email, age, role, dept, mgr, start) { }
```
**After:**
```typescript
function createUser(options: CreateUserOptions) { }

interface CreateUserOptions {
  name: string;
  email: string;
  age?: number;
  role?: string;
  department?: string;
  manager?: string;
  startDate?: Date;
}
```

## Complexity Report Format

```markdown
### Complexity Analysis

| Metric | Value | Assessment |
|--------|-------|------------|
| LOC Added | ~X | [Small/Medium/Large] |
| LOC Removed | ~X | - |
| Net Change | +X | - |
| Complexity Rating | [Low/Medium/High] | [Justification] |

### Identified Hotspots

| Location | Issue | Complexity | Suggestion |
|----------|-------|------------|------------|
| `file.ts:45-90` | 45-line function | High | Extract validation logic |
| `file.ts:100-150` | 4-level nesting | High | Use early returns |
| `file.ts:200` | 6 parameters | Medium | Use options object |
```

## Quick Reference

| Smell | Threshold | Fix |
|-------|-----------|-----|
| Long function | > 50 lines | Extract method |
| Deep nesting | > 3 levels | Early returns |
| Many params | > 4 | Options object |
| Long conditionals | > 3 conditions | Extract to function |
| Duplicate code | 3+ occurrences | Extract helper |
| God class | > 300 lines | Split responsibilities |
