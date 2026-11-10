# Story — Rational Exponent Boundary Fix

**ID:** PER-171  
**Type:** Story  
**Epic:** N/A (standalone bug fix)  
**Track:** Quick  
**Estimate:** 2 points  

---

## Description

Fix `exponent(::Rational)` to return correct results for rationals near power-of-two boundaries. Currently `exponent((2^60 - 1) // 2^60)` returns `0` instead of `-1` because the generic `Real` fallback converts to `Float64`, and `Float64((2^60 - 1) // 2^60) == 1.0` crosses the boundary. Implement an integer-only method that computes the exponent using numerator and denominator arithmetic without floating-point conversion.

---

## Technical Specification

### Implementation Location

**File to modify:** `src/builtins.c` (or appropriate Julia base source file for `exponent` methods)  
**Test file:** `test/math.jl`

### Algorithm

The exponent of a rational `p // q` is `floor(log2(abs(p // q)))`, computed as:

```
exponent(r::Rational) = exponent(numerator(r)) - exponent(denominator(r)) - (abs(numerator(r)) < abs(denominator(r)) ? 1 : 0)
```

This uses the existing `exponent(::Integer)` (fixed in #59508) and adjusts for the division. When `|p| < |q|`, the result is negative; the `-1` accounts for the floor of a negative log.

**Edge cases:**
- `exponent(1//1)` → `0 - 0 - 0 = 0` ✓
- `exponent(2//1)` → `1 - 0 - 0 = 1` ✓
- `exponent((2^60 - 1) // 2^60)` → `59 - 60 - 0 = -1` ✓ (since `2^60 - 1 < 2^60`)

### Files Changed

1. **`src/builtins.c`** (or the file containing `exponent` implementations):
   - Add new method signature for `Rational` input
   - Implement integer-only logic as specified above
   - Do NOT modify existing `exponent(::AbstractFloat)` or `exponent(::Integer)` methods

2. **`test/math.jl`**:
   - Add regression test block for `Rational` exponent edge cases
   - Test the boundary case `(2^60 - 1) // 2^60`
   - Test the invariant `2^exponent(q) <= q < 2^(exponent(q)+1)`
   - Test edge cases `1//1` and `2//1`

### Constraints

- **Integer arithmetic only** — no conversion to `Float64` or any floating-point type
- **Scope:** `Rational` inputs only; `Float` and `Integer` behavior unchanged
- **Existing tests must pass** — `./usr/bin/julia --startup-file=no test/runtests.jl math`
- **Whitespace compliance** — `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl`

---

## Acceptance Criteria

### AC1: Boundary case returns correct exponent
**Given** the rational `q = (2^60 - 1) // 2^60`  
**When** `exponent(q)` is called  
**Then** it returns `-1` (not `0`)

**Verify:**
```julia
@test exponent((2^60 - 1) // 2^60) == -1
```

---

### AC2: Exponent invariant holds
**Given** the rational `q = (2^60 - 1) // 2^60`  
**When** computing `2^exponent(q)`  
**Then** `2^exponent(q) <= q` is true

**Verify:**
```julia
q = (2^60 - 1) // 2^60
@test 2^exponent(q) <= q
```

---

### AC3: Edge case — unity
**Given** the rational `1//1`  
**When** `exponent(1//1)` is called  
**Then** it returns `0`

**Verify:**
```julia
@test exponent(1//1) == 0
```

---

### AC4: Edge case — power of two
**Given** the rational `2//1`  
**When** `exponent(2//1)` is called  
**Then** it returns `1`

**Verify:**
```julia
@test exponent(2//1) == 1
```

---

### AC5: Math test suite passes
**Given** the changes to `src/builtins.c` and `test/math.jl`  
**When** the math test suite is run with `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all tests pass with exit code `0`

**Verify:**  
CI step; manual verification: `./usr/bin/julia --startup-file=no test/runtests.jl math` exits `0`

---

### AC6: Whitespace checker passes
**Given** the modified source files  
**When** the whitespace checker runs with `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl`  
**Then** no whitespace violations are reported

**Verify:**  
CI step; manual verification: `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl` exits `0`

---

## Notes

- **Reference:** Integer `exponent` was fixed in PR #59508; this extends the fix to `Rational`
- **Float64 precision loss:** The bug occurs because `Float64((2^60 - 1) // 2^60)` rounds to `1.0`, losing the distinction between a value just below 1 and 1 itself
- **Design decision:** The integer-only approach avoids all floating-point conversion and guarantees correct results for arbitrarily large rationals within integer precision limits

---

## Dependencies

**Blocking:** None  
**Blocked by:** None

---

## Story Points Justification

**2 points** — Single-method implementation with clear algorithm, self-contained test additions, no architecture changes. Quick track fits: isolated bug fix with explicit acceptance criteria and known solution approach.
