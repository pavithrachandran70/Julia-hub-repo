# Story — PER-187

**Title:** Fix exponent() for Rational to avoid float conversion and return correct power-of-two exponent

**Type:** story  
**ID:** 187  
**Epic:** (standalone)  
**Track:** quick  
**Effort:** 1 point  
**Risk:** low  
**Assignee:** (unassigned)  
**Status:** planned

---

## Description

The `exponent()` function for `Rational` numbers currently uses a generic fallback that converts to `Float64`, causing incorrect results when the rational is just below a power of two. For example, `exponent((2^60 - 1) // 2^60)` returns `0` instead of `-1` because the float conversion crosses the power-of-two boundary (`Float64((2^60 - 1) // 2^60) == 1.0`).

This story implements a direct integer-based `exponent()` method for `Rational` that computes the result using the numerator and denominator exponents without float conversion, ensuring `2^exponent(q) <= abs(q) < 2^(exponent(q)+1)` holds for all rational `q`.

---

## Technical Specification

### Algorithm

Compute the exponent for a Rational using integer arithmetic only by leveraging the existing integer `exponent()` methods (already fixed per #59508). The implementation computes `exponent(abs(numerator)) - exponent(denominator)` to produce the power-of-two floor directly, avoiding float conversion entirely. The result is the largest integer `y` such that `2^y <= abs(q)`.

### Implementation Location

**Repository tree mismatch identified:** The current repository at `/home/pavc/Documents/Julia-Repo` contains only C/C++ source files under `src/` and a `README.md`. The repo tree shows no Julia `.jl` files. The ticket specifies "Repo Julia only" and the test commands reference Julia tooling (`./usr/bin/julia --startup-file=no test/runtests.jl math`), but the existing tree does not contain Julia base source code or test files.

**Expected files in a full Julia repository checkout:**
- Implementation: `base/intfuncs.jl` (where other integer exponent methods are defined) or `base/rational.jl` (where Rational-specific methods live)
- Tests: `test/math.jl`

**Action required:** The implementer must work from a full clone of the upstream Julia repository (https://github.com/JuliaLang/julia) where `base/` and `test/` directories exist, or confirm the correct repository path if the checkout at `/home/pavc/Documents/Julia-Repo` is incomplete or misconfigured. Cannot proceed with implementation until Julia source files are accessible.

### Edge Cases Handled

1. **Rationals just below 1:** `(2^60 - 1) // 2^60` returns `-1` (not `0`)
2. **Exact powers of two:** `1//1` returns `0`, `2//1` returns `1`
3. **Negative rationals:** Use `abs()` before computing exponent (sign doesn't affect power-of-two floor)
4. **Denominators > 1:** Subtracting denominator exponent correctly shifts the result down

---

## Acceptance Criteria

### AC-1: exponent((2^60 - 1) // 2^60) == -1
**Given** the Rational `q = (2^60 - 1) // 2^60`  
**When** `exponent(q)` is called  
**Then** the function returns `-1`

**Verification:** Unit test assertion in test/math.jl

---

### AC-2: 2^exponent(q) <= q for that rational
**Given** the Rational `q = (2^60 - 1) // 2^60`  
**When** `exponent(q)` is evaluated  
**Then** `2^exponent(q) <= q` is true

**Verification:** Unit test assertion in test/math.jl

---

### AC-3: exponent(1//1) == 0 and exponent(2//1) == 1
**Given** the Rationals `1//1` and `2//1`  
**When** `exponent(1//1)` and `exponent(2//1)` are called  
**Then** they return `0` and `1` respectively

**Verification:** Unit test assertions in test/math.jl

---

### AC-4: ./usr/bin/julia --startup-file=no test/runtests.jl math passes
**Given** the implementation is complete  
**When** the test suite is run with `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all tests pass

**Verification:** CI/local test run

---

### AC-5: ./usr/bin/julia --startup-file=no contrib/check-whitespace.jl passes
**Given** the implementation is complete  
**When** `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl` is run  
**Then** the check passes with no warnings

**Verification:** CI/local lint run

---

## Requirements Coverage

- **REQ-1** (exponent((2^60 - 1) // 2^60) returns -1): Covered by AC-1
- **REQ-2** (2^exponent(q) <= q for the regression case): Covered by AC-2
- **REQ-3** (Handle exact powers of two correctly): Covered by AC-3
- **REQ-4** (Pass existing math test suite): Covered by AC-4
- **REQ-5** (Pass whitespace validation): Covered by AC-5

---

## Tests

This story adds 5 test assertions to the existing math test suite (exact count matches the 5 acceptance criteria):

1. `@test exponent((2^60 - 1) // 2^60) == -1` (AC-1)
2. `@test 2^exponent((2^60 - 1) // 2^60) <= (2^60 - 1) // 2^60` (AC-2)
3. `@test exponent(1//1) == 0` (AC-3)
4. `@test exponent(2//1) == 1` (AC-3)
5. Full test/math.jl suite passes (AC-4)

AC-5 (whitespace check passes) is verified by running the linter but adds no new test assertions.

**Test file location:** test/math.jl (in a full Julia repository checkout; not present in current repo tree at /home/pavc/Documents/Julia-Repo)

---

## Implementation Notes

### Files to Modify

**Prerequisite:** A full Julia repository checkout is required. The current tree at `/home/pavc/Documents/Julia-Repo` contains only C/C++ source and lacks Julia base code and tests.

**In a full Julia repository:**
1. **Implementation file:** `base/intfuncs.jl` or `base/rational.jl` — Add the new `exponent(::Rational)` method using integer arithmetic
2. **Test file:** `test/math.jl` — Add the 5 test assertions listed above

### Implementation Approach

1. Add a method for `exponent(::Rational)` that computes `exponent(abs(numerator)) - exponent(denominator)` using existing integer exponent methods
2. Add the 5 test assertions to test/math.jl
3. Run `./usr/bin/julia --startup-file=no test/runtests.jl math` to verify no regressions
4. Run `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl` to verify formatting

### Dependencies

- None (uses existing integer `exponent()` methods fixed in #59508)

---

## Constraints

- **Scope:** Julia repository only; do not modify `exponent()` for Float or Integer types (those are already fixed)
- **No float conversion:** The implementation must compute the result using integer arithmetic only
- **Backward compatibility:** Existing behavior for other types (Float, Integer) must not change

---

## Open Questions

**Q1:** The repository tree at `/home/pavc/Documents/Julia-Repo` contains only C/C++ source files under `src/` and no Julia `.jl` files. Is this a partial checkout, a fork with a different structure, or should the implementer work from a different repository path?

**Resolution needed before implementation can begin.**

---

## Notes

- The ticket specifies "Repo Julia only" and references Julia test commands, but the repo tree shows no Julia source code
- All five acceptance criteria from the ticket are preserved with exact wording
- The fix is self-contained in one method and its tests once the correct repository is accessed
- The regression case is explicitly provided in the ticket
- The algorithm avoids float conversion by using integer exponents directly
