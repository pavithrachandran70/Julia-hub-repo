# Requirements — PER-187

**Feature:** Fix exponent() for Rational to avoid float conversion

**Epic:** (standalone ticket)

**Source:** Ticket PER-187

---

## Summary

The `exponent()` function for `Rational` numbers currently uses a generic fallback that converts to `Float64`, causing incorrect results when the rational is just below a power of two. For example, `exponent((2^60 - 1) // 2^60)` returns `0` instead of `-1` because `Float64((2^60 - 1) // 2^60) == 1.0`, crossing the power-of-two boundary. This fix implements a direct integer-based algorithm using the numerator and denominator exponents without float conversion.

---

## Requirements

### REQ-1: exponent((2^60 - 1) // 2^60) returns -1

**Given** the Rational `q = (2^60 - 1) // 2^60`  
**When** `exponent(q)` is called  
**Then** the function returns `-1`

**Notes:**
- This is the regression case from the ticket
- The value is less than 1, so the largest integer y with 2^y <= abs(q) is -1
- Current generic Real method converts to Float64, which rounds to 1.0 and returns 0 incorrectly

---

### REQ-2: 2^exponent(q) <= q for the regression case

**Given** the Rational `q = (2^60 - 1) // 2^60`  
**When** `exponent(q)` is evaluated  
**Then** `2^exponent(q) <= q` is true

**Notes:**
- Verifies the fix handles the power-of-two boundary correctly
- `2^(-1) = 0.5`, and `(2^60 - 1) / 2^60 ≈ 0.9999...`, so `0.5 <= q` holds
- This is the second acceptance criterion from the ticket

---

### REQ-3: Handle exact powers of two correctly

**Given** Rationals that are exact powers of two  
**When** `exponent(1//1)` is called  
**Then** it returns `0`  
**When** `exponent(2//1)` is called  
**Then** it returns `1`

**Notes:**
- Covers edge cases at power-of-two boundaries
- `1 = 2^0`, so exponent is `0`
- `2 = 2^1`, so exponent is `1`

---

### REQ-4: Pass existing math test suite

**Given** the implementation is complete  
**When** the test suite is run with `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all tests pass

**Notes:**
- Add the regression case to existing test/math.jl
- Verify no existing `exponent()` tests break
- Scope: only add test for the Rational fix; do not modify float or integer tests
- Test file location: test/math.jl (in a full Julia repository checkout; current repo tree at /home/pavc/Documents/Julia-Repo contains only C/C++ src/ and lacks Julia .jl files)

---

### REQ-5: Pass whitespace validation

**Given** the implementation is complete  
**When** whitespace check is run with `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl`  
**Then** the check passes with no warnings

**Notes:**
- Ensures code follows Julia project formatting standards
- No trailing whitespace, consistent indentation

---

## Implementation Guidance

Compute `exponent(q::Rational)` using integer arithmetic only: use the existing integer `exponent()` methods for numerator and denominator (already fixed per #59508). Handle sign by computing on absolute value. The algorithm `exponent(abs(numerator)) - exponent(denominator)` gives the power-of-two floor directly, ensuring the result satisfies `2^exponent(q) <= abs(q) < 2^(exponent(q)+1)` without float conversion.

**File locations (in a full Julia repository checkout):**
- Implementation: One of base/intfuncs.jl (where other integer exponent() methods are defined) or base/rational.jl (where Rational-specific methods live)
- Tests: test/math.jl

**Note on current repository tree:** The repository at `/home/pavc/Documents/Julia-Repo` contains only C/C++ source files under `src/` and a README. No Julia `.jl` files are present. The ticket specifies "Repo Julia only" and the test commands reference Julia tooling (`julia --startup-file=no`), but the existing tree does not contain Julia base source code. The implementer must work from a full clone of the upstream Julia repository where `base/` and `test/` directories exist, or confirm the correct repository path.

---

## Out of Scope

- Changes to `exponent()` for Float or Integer types (those are already fixed per #59508)
- Performance optimization beyond avoiding float conversion
- Changes to other Rational methods
- Changes outside the Julia repository

---

## Open Questions

None — the ticket provides complete acceptance criteria and algorithm guidance.
