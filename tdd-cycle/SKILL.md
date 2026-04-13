---
name: tdd-cycle
description: Use before writing any implementation code — walks through the full TDD cycle: write the failing test first, watch it fail, then write just enough code to pass it
---

# TDD Cycle

## Overview

Writing implementation before tests leads to untestable code, over-engineering, and false confidence. This skill enforces the strict RED → GREEN → REFACTOR cycle so tests drive the design.

**Core principle:** If there's no failing test, you're not allowed to write implementation code.

**Announce at start:** "I'm using the tdd-cycle skill. Writing failing test first."

## When to Use

Use for ANY new functionality:
- New features
- Bug fixes (write a test that reproduces the bug first)
- New API endpoints
- New components
- Utility functions
- Database queries

**Use this ESPECIALLY when:**
- You're tempted to "just write the code first"
- The feature feels simple (simple features still need tests)
- You're fixing a bug without a reproduction test
- You're refactoring existing code

## The Iron Rule

```
NO IMPLEMENTATION CODE WITHOUT A FAILING TEST FIRST
```

If you haven't seen the test fail with the right error, you cannot write production code.

## The Cycle

### Phase 1 — RED (Write a failing test)

1. Write the **smallest possible test** for the next behavior
2. Run it — confirm it **FAILS** with the expected reason
3. Read the error message — it should describe what's missing, not a syntax error

```typescript
// Example: Testing a function that doesn't exist yet
describe('formatCurrency', () => {
  it('formats USD correctly', () => {
    expect(formatCurrency(1234.56, 'USD')).toBe('$1,234.56')
  })
})
// Run: FAIL — formatCurrency is not defined ✓
```

### Phase 2 — GREEN (Write minimal implementation)

1. Write the **minimum code** to make the test pass
2. Do NOT over-engineer — just make the test green
3. Run tests — confirm they **PASS**

```typescript
// Minimal implementation — just enough to pass
export function formatCurrency(amount: number, currency: string): string {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(amount)
}
// Run: PASS ✓
```

### Phase 3 — REFACTOR (Clean up)

1. Remove duplication
2. Improve naming
3. Extract if needed
4. Tests must still pass after every change

```typescript
// Refactored: extract formatter config
const FORMATTERS: Record<string, Intl.NumberFormat> = {}

export function formatCurrency(amount: number, currency: string): string {
  if (!FORMATTERS[currency]) {
    FORMATTERS[currency] = new Intl.NumberFormat('en-US', { style: 'currency', currency })
  }
  return FORMATTERS[currency].format(amount)
}
// Run: PASS ✓ (cached formatter, same behavior)
```

## Test Quality Rules

### Good Test Checklist
- [ ] Tests one behavior (not multiple concerns)
- [ ] Test name describes expected behavior: `it('returns null when user not found')`
- [ ] Arrange / Act / Assert structure
- [ ] No implementation details in test (test behavior, not internals)
- [ ] Can run in any order (no test interdependence)
- [ ] Fast (no real network/DB calls in unit tests — use mocks)

### Test Naming Pattern
```
[unit under test] [action/condition] [expected outcome]
'formatCurrency when amount is negative returns negative formatted string'
'UserService.findById when user does not exist returns null'
'POST /api/users when email already taken returns 409 Conflict'
```

## Coverage Targets

| Layer | Minimum | Goal |
|-------|---------|------|
| Domain logic / utils | 90% | 100% |
| API routes | 80% | 90% |
| UI components | 70% | 80% |
| Integration tests | Key flows covered | — |

## Test Pyramid

```
        /\
       /E2E\         ← Few, critical user journeys
      /------\
     /Integr. \      ← API + DB flows
    /----------\
   /  Unit Tests \   ← Many, fast, isolated
  /--------------\
```

Write more unit tests than integration tests. Write more integration tests than E2E.

## Bug Fix Protocol

**NEVER fix a bug without writing a reproduction test first:**

1. Write a test that fails because of the bug
2. Confirm test fails (RED)
3. Fix the bug
4. Confirm test passes (GREEN)
5. This prevents regressions forever

```typescript
// Bug: formatCurrency crashes when amount is 0
it('handles zero amount', () => {
  expect(() => formatCurrency(0, 'USD')).not.toThrow()
  expect(formatCurrency(0, 'USD')).toBe('$0.00')
})
// Run: FAIL (reproduces bug) → Fix → PASS
```

## Activation Phrases

Claude activates this skill when you say:
- "implement feature..." / "add function..."
- "just start implementing..."
- "write the code for..."
- "fix this bug" (without a test)
