# Requirements — PER-118

## Given/When/Then Acceptance Criteria

### AC-1: Rational just below 1 returns correct exponent
**Given** a `Rational` value `(2^60 - 1) // 2^60` (just below 1.0)  
**When** `exponent()` is called on this value  
**Then** it returns `-1`, not `0`

**Verification:** Direct assertion in `test/math.jl`

---

### AC-2: Exponent invariant holds for rationals below 1
**Given** a `Rational` value `q = (2^60 - 1) // 2^60`  
**When** computing `2^exponent(q)`  
**Then** the invariant `2^exponent(q) <= q` holds true

**Verification:** Assertion in `test/math.jl` checking the invariant for this specific test case

---

### AC-3: Exponent of unity rational is zero
**Given** a `Rational` value `1//1`  
**When** `exponent()` is called  
**Then** it returns `0`

**Verification:** Regression test in `test/math.jl`

---

### AC-4: Exponent of power-of-two rational is correct
**Given** a `Rational` value `2//1`  
**When** `exponent()` is called  
**Then** it returns `1`

**Verification:** Regression test in `test/math.jl`

---

### AC-5: Math test suite passes
**Given** the implementation is complete  
**When** running `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all math tests pass, including the new `exponent(::Rational)` tests

**Verification:** CI or local test run confirms zero failures

---

### AC-6: Whitespace check passes
**Given** the implementation is complete  
**When** running `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl`  
**Then** no whitespace violations are reported

**Verification:** CI or local lint run confirms zero issues

---

## Scope Constraints

- **In scope:**
  - Add specialized `exponent(::Rational)` method in `base/rational.jl`
  - Use integer-only arithmetic (no float conversion)
  - Add regression tests to existing `test/math.jl`
  - Ensure the exponent invariant `2^exponent(q) <= q < 2^(exponent(q)+1)` holds

- **Out of scope:**
  - Changes to `exponent` for `Float64`, `Int`, or other types
  - Changes outside the `Julia` repo
  - New test files or test infrastructure
  - Performance optimization beyond correctness

---

## Technical Notes

- **Root cause:** The generic `Real` fallback converts `Rational` to `Float64`, causing precision loss at power-of-two boundaries
- **Fix:** Implement `exponent(q::Rational)` using integer operations on `q.num` and `q.den`
- **Test location:** `test/math.jl` (existing file)
- **Implementation location:** `base/rational.jl` (existing file)

---

## Open Questions

None — all acceptance criteria are concrete and testable as stated.
