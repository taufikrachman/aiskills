# Refactoring — Safe Code Improvement

Systematically improve code structure without changing external behavior.

## Rules

### 1. Refactoring vs Rewriting
```
Refactor: improve internal structure, behavior unchanged (safe)
Rewrite:  throw away and rebuild (risky)
```
- Refactor in small, testable steps. Never do a "big bang" rewrite.
- If you need to change behavior AND improve structure, do it in TWO commits.

### 2. Safety Nets
- BEFORE refactoring: ensure tests pass. If no tests, write characterization tests.
- Run tests after EVERY small refactoring step. Not at the end.
- Use IDE refactoring tools (Rename, Extract Method, Move) — they're safer than manual edits.
- Commit after each safe step. Easy to `git revert` if something breaks.

### 3. Common Refactoring Patterns

**Extract Method** (most common):
```ts
// Before: long method doing multiple things
function processOrder(order) {
  // 20 lines of validation
  // 30 lines of calculation
  // 15 lines of notification
}

// After: extracted into focused methods
function processOrder(order) {
  validateOrder(order);
  const total = calculateTotal(order);
  notifyCustomer(order, total);
}
```

**Replace Magic Numbers with Constants:**
```ts
// Before
if (status === 3) { ... }
setTimeout(fn, 900000);

// After
const STATUS_EXPIRED = 3;
const TOKEN_TTL_MS = 15 * 60 * 1000;
if (status === STATUS_EXPIRED) { ... }
setTimeout(fn, TOKEN_TTL_MS);
```

**Early Return (guard clauses):**
```ts
// Before: nested conditionals
function getDiscount(user) {
  if (user) {
    if (user.isPremium) {
      if (user.subscriptionActive) {
        return 0.2;
      }
    }
  }
  return 0;
}

// After: flat, readable
function getDiscount(user) {
  if (!user) return 0;
  if (!user.isPremium) return 0;
  if (!user.subscriptionActive) return 0;
  return 0.2;
}
```

**Replace Conditional with Polymorphism:**
```ts
// Before
function getFee(method: string) {
  if (method === 'credit_card') return 0.03;
  if (method === 'bank_transfer') return 0.01;
  if (method === 'qris') return 0.007;
}

// After
const feeMap = { credit_card: 0.03, bank_transfer: 0.01, qris: 0.007 };
function getFee(method: string) { return feeMap[method] ?? 0; }
```

### 4. When to Refactor
- ✅ Before adding a feature (make the change easy, then make the easy change)
- ✅ After fixing a bug (clean up the messy code that caused it)
- ✅ During code review (small improvements suggested by reviewer)
- ❌ During a bug hunt (separate the fix from the clean-up)
- ❌ Right before a deadline (risk of introducing new bugs)
- ❌ Without test coverage (too risky)

### 5. Code Smells to Target
| Smell | Refactoring |
|-------|------------|
| Long method (>50 lines) | Extract Method |
| Duplicated code | Extract shared function |
| Too many parameters (>4) | Introduce Parameter Object |
| Primitive obsession | Replace with Value Object |
| Switch/if-else chains | Replace with map/polymorphism |
| Comments explaining WHAT | Rename to make code self-documenting |
| Dead code | Delete it (git history has it if needed) |

### 6. Commit Strategy
```bash
# GOOD: small, safe steps
git commit -m "refactor: extract validateOrder method"
git commit -m "refactor: replace magic numbers with constants"
git commit -m "refactor: flatten nested conditionals with guard clauses"

# BAD: one giant commit
git commit -m "refactor everything"
```

## Anti-Patterns
- ❌ Refactoring without tests (you will break something)
- ❌ Changing behavior during refactoring (that's a feature, separate commit)
- ❌ "Cleaning up" while fixing a bug (separate concerns)
- ❌ Big-bang refactors (small, safe steps only)
- ❌ Skipping code review on refactored code
