# PER-118 — `exponent` of a `Rational` just below 1 returns 0 (LLD)

**Type:** story  
**Status:** draft  
**Epic:** (none — standalone backlog story)  
**ID:** 118  
**Repository:** Julia  
**hitl:** false

---

## Summary

`exponent((2^60 - 1) // 2^60)` returns `0` instead of `-1` because the generic `Real` method converts the `Rational` to `Float64`, which rounds up to `1.0`, crossing the power-of-two boundary. This violates the exponent invariant `2^exponent(q) <= q < 2^(exponent(q)+1)`. The fix is to give `Rational` its own `exponent` method that computes the result using integer arithmetic on `num` and `den` without converting to float.

---

## Acceptance Criteria

### AC-1: Rational just below 1 returns correct exponent
**Given** a `Rational` value `(2^60 - 1) // 2^60` (just below 1.0)  
**When** `exponent()` is called on this value  
**Then** it returns `-1`, not `0`

---

### AC-2: Exponent invariant holds for rationals below 1
**Given** a `Rational` value `q = (2^60 - 1) // 2^60`  
**When** computing `2^exponent(q)`  
**Then** the invariant `2^exponent(q) <= q` holds true

---

### AC-3: Exponent of unity rational is zero
**Given** a `Rational` value `1//1`  
**When** `exponent()` is called  
**Then** it returns `0`

---

### AC-4: Exponent of power-of-two rational is correct
**Given** a `Rational` value `2//1`  
**When** `exponent()` is called  
**Then** it returns `1`

---

### AC-5: Math test suite passes
**Given** the implementation is complete  
**When** running `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all math tests pass, including the new `exponent(::Rational)` tests

---

### AC-6: Whitespace check passes
**Given** the implementation is complete  
**When** running `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl`  
**Then** no whitespace violations are reported

---

## File Plan

### Files to Modify

**1. `base/rational.jl`**
- **Purpose:** Add specialized `exponent(::Rational)` method
- **Location:** Near existing mathematical operations on `Rational` types (likely near `float`, `sign`, or other numeric conversion methods)
- **Changes:**
  - Add new method definition: `exponent(q::Rational)`
  - No imports needed (uses only `Base` functions: `iszero`, `numerator`, `denominator`, `abs`, `ndigits`)

**2. `test/math.jl`**
- **Purpose:** Add regression tests for `exponent(::Rational)`
- **Location:** Existing test file under `test/` directory
- **Changes:**
  - Add new `@testset` block for `exponent(::Rational)` regression tests
  - Contains 4 test assertions (AC-1 through AC-4)

---

## Interfaces

### New Method Signature

```julia
exponent(q::Rational) -> Int
```

**Parameters:**
- `q::Rational` — The rational number to compute the exponent for

**Returns:**
- `Int` — The floor of log₂(abs(q)), i.e., the largest integer `n` such that `2^n <= abs(q)`
- Special case: `-Inf` when `q == 0` (matches existing `exponent` behavior for other types)

**Behavior:**
- Computes the exponent using integer-only arithmetic on the numerator and denominator
- Does **not** convert to `Float64` at any point
- Maintains the invariant: `2^exponent(q) <= abs(q) < 2^(exponent(q)+1)`

**Integration:**
- Overrides the generic `Real` fallback for `Rational` types
- Does not affect `exponent` behavior for `Float64`, `Int`, or other numeric types
- Uses existing Julia `Base` functions only; no new dependencies

---

## Algorithm Design

### Integer-Only Exponent Computation

**Mathematical Foundation:**
```
exponent(q) = floor(log₂(abs(q)))
            = floor(log₂(abs(num/den)))
            = floor(log₂(abs(num)) - log₂(den))
            = floor(log₂(abs(num))) - floor(log₂(den))  [when den > 0]
```

**Implementation Strategy:**

1. **MSB Position Function:**
   - For an integer `n > 0`, the position of the most significant bit (MSB) equals `floor(log₂(n))`
   - In Julia: `ndigits(n, base=2) - 1` computes this efficiently
   - Example: `ndigits(8, base=2) - 1 = 4 - 1 = 3`, and indeed `floor(log₂(8)) = 3`

2. **Algorithm Steps:**
   ```julia
   function exponent(q::Rational)
       # 1. Handle zero case
       iszero(q) && return -Inf
       
       # 2. Compute MSB positions
       num_exp = ndigits(abs(numerator(q)), base=2) - 1
       den_exp = ndigits(denominator(q), base=2) - 1
       
       # 3. Subtract to get log difference
       return num_exp - den_exp
   end
   ```

3. **Edge Cases:**
   - **Zero rational:** Return `-Inf` (consistent with `exponent(0.0)`)
   - **Negative rational:** Use `abs(numerator(q))` to work with magnitude only
   - **num < den (|q| < 1):** Result is negative (MSB of num is at lower position than MSB of den)
   - **num == den (|q| == 1):** Result is `0` (MSB positions are equal)
   - **num > den (|q| > 1):** Result is positive (MSB of num is at higher position than MSB of den)

4. **Correctness Verification:**
   - For `q = (2^60 - 1) // 2^60`:
     - `num_exp = ndigits(2^60 - 1, base=2) - 1 = 60 - 1 = 59`
     - `den_exp = ndigits(2^60, base=2) - 1 = 61 - 1 = 60`
     - `exponent(q) = 59 - 60 = -1` ✓
   - For `q = 1//1`:
     - `num_exp = ndigits(1, base=2) - 1 = 1 - 1 = 0`
     - `den_exp = ndigits(1, base=2) - 1 = 1 - 1 = 0`
     - `exponent(q) = 0 - 0 = 0` ✓
   - For `q = 2//1`:
     - `num_exp = ndigits(2, base=2) - 1 = 2 - 1 = 1`
     - `den_exp = ndigits(1, base=2) - 1 = 1 - 1 = 0`
     - `exponent(q) = 1 - 0 = 1` ✓

---

## Implementation Details

### Code Placement in `base/rational.jl`

**Location:** Insert after existing mathematical operations (likely near line ~100-150, after basic arithmetic operators but before advanced functions like `gcd` or conversion methods)

**Code Block:**
```julia
"""
    exponent(q::Rational) -> Int

Compute the binary exponent of a rational number using integer arithmetic.
Returns the largest integer `n` such that `2^n <= abs(q)`.

# Examples
```jldoctest
julia> exponent(1//1)
0

julia> exponent(2//1)
1

julia> exponent((2^60 - 1) // 2^60)
-1
```
"""
function exponent(q::Rational)
    iszero(q) && return -Inf
    num_exp = ndigits(abs(numerator(q)), base=2) - 1
    den_exp = ndigits(denominator(q), base=2) - 1
    return num_exp - den_exp
end
```

**Notes:**
- Follows Julia docstring conventions with example usage
- No explicit imports needed (all functions are in `Base`)
- No type conversions to floating-point types
- Sign-agnostic via `abs(numerator(q))`

---

### Test Block in `test/math.jl`

**Location:** Insert in the appropriate section of `test/math.jl` (likely near other numeric function tests, around line ~500-1000)

**Code Block:**
```julia
@testset "exponent(::Rational) regression" begin
    # AC-1: Rational just below 1 returns correct exponent
    @test exponent((2^60 - 1) // 2^60) == -1
    
    # AC-2: Exponent invariant holds for rationals below 1
    q = (2^60 - 1) // 2^60
    @test 2^exponent(q) <= q
    
    # AC-3: Exponent of unity rational is zero
    @test exponent(1//1) == 0
    
    # AC-4: Exponent of power-of-two rational is correct
    @test exponent(2//1) == 1
end
```

**Test Coverage:**
1. **AC-1:** Direct test of the failing case
2. **AC-2:** Invariant verification for the failing case
3. **AC-3:** Unity rational baseline
4. **AC-4:** Power-of-two rational baseline

---

## Tests

### Test 1: Edge case — rational just below 1
**Maps to:** AC-1  
**Verification Approach:** Direct assertion in `test/math.jl`
```julia
@test exponent((2^60 - 1) // 2^60) == -1
```

---

### Test 2: Exponent invariant
**Maps to:** AC-2  
**Verification Approach:** Assertion in `test/math.jl` checking the invariant for the specific test case
```julia
q = (2^60 - 1) // 2^60
@test 2^exponent(q) <= q
```

---

### Test 3: Exponent of unity
**Maps to:** AC-3  
**Verification Approach:** Regression test in `test/math.jl`
```julia
@test exponent(1//1) == 0
```

---

### Test 4: Exponent of power-of-two rational
**Maps to:** AC-4  
**Verification Approach:** Regression test in `test/math.jl`
```julia
@test exponent(2//1) == 1
```

---

### Test 5: Full math test suite
**Maps to:** AC-5  
**Verification Approach:** CI or local test run confirms zero failures
```bash
./usr/bin/julia --startup-file=no test/runtests.jl math
```

---

### Test 6: Whitespace validation
**Maps to:** AC-6  
**Verification Approach:** CI or local lint run confirms zero issues
```bash
./usr/bin/julia --startup-file=no contrib/check-whitespace.jl
```

---

## Scope Constraints

**In scope:**
- Single method addition to `base/rational.jl`
- Regression tests added to existing `test/math.jl`
- Integer-only arithmetic (no float conversion)
- Validation of the exponent invariant for the failing case

**Out of scope:**
- Changes to `exponent` for `Float64`, `Int`, or other numeric types
- New test infrastructure or test files
- Performance optimization beyond correctness
- Changes outside the `Julia` repository

---

## Notes

- The integer-only algorithm avoids the precision loss that occurs when converting `Rational` to `Float64`
- The algorithm leverages the fact that `log₂(a/b) = log₂(a) - log₂(b)`, and for integers, `log₂(n)` is the position of the most significant bit
- `ndigits(n, base=2) - 1` gives the floor of `log₂(n)` for positive integers, which is exactly the exponent
- The fix is minimal and localized to `base/rational.jl`, with tests in the existing `test/math.jl` file
- No new dependencies or imports required; uses only existing `Base` functions
- Maintains consistency with existing `exponent` behavior for other numeric types (special handling of zero, sign-agnostic via `abs`)

---

## References

- Ticket: PER-118
- Requirements: `requirements.md`
- Breakdown: `plan.md`
- Related issue: #59508 (fixed integer exponent, but not rational exponent)
