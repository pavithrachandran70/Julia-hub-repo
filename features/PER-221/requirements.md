# requirements.md

## Overview

This requirements document specifies the acceptance criteria for fixing the unsigned integer wrap-around bug in `Base.Checked.checked_pow` where `checked_pow(typemax(UInt), -1)` incorrectly returns the input instead of throwing a `DomainError`.

## Given/When/Then Acceptance Criteria

### AC-1: Reject typemax for unsigned integers with negative exponents

**Given** an unsigned integer type `T` where `T` is `UInt8`, `UInt64`, or `UInt128`  
**When** `checked_pow(typemax(T), -1)` is called  
**Then** a `DomainError` is thrown

**Verification:** Call `checked_pow(typemax(UInt8), -1)`, `checked_pow(typemax(UInt64), -1)`, and `checked_pow(typemax(UInt128), -1)` and verify each throws `DomainError`. (Three test cases in `test/math.jl`)

---

### AC-2: Reject other unsigned integers with negative exponents

**Given** an unsigned integer value that is not `typemax`  
**When** `checked_pow(UInt8(2), -1)` is called  
**Then** a `DomainError` is thrown

**Verification:** Call `checked_pow(UInt8(2), -1)` and verify it throws `DomainError`. (One test case in `test/math.jl`)

---

### AC-3: Preserve positive exponent behavior for unsigned integers

**Given** an unsigned integer with a positive exponent  
**When** `checked_pow(UInt8(2), 2)` is called  
**Then** the result is `0x04`

**Verification:** Call `checked_pow(UInt8(2), 2)` and verify the result equals `0x04`. (One test case in `test/math.jl`)

---

### AC-4: Preserve signed -1 behavior with negative exponents (odd)

**Given** a signed integer value of `-1` and an odd negative exponent  
**When** `checked_pow(Int8(-1), -1)` is called  
**Then** the result is `Int8(-1)`

**Verification:** Call `checked_pow(Int8(-1), -1)` and verify the result equals `Int8(-1)`. (One test case in `test/math.jl`)

---

### AC-5: Preserve signed -1 behavior with negative exponents (even)

**Given** a signed integer value of `-1` and an even negative exponent  
**When** `checked_pow(Int8(-1), -2)` is called  
**Then** the result is `Int8(1)`

**Verification:** Call `checked_pow(Int8(-1), -2)` and verify the result equals `Int8(1)`. (One test case in `test/math.jl`)

---

### AC-6: Math test suite passes

**Given** the fix is implemented  
**When** the math test suite is executed with `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all tests pass

**Verification:** Run `./usr/bin/julia --startup-file=no test/runtests.jl math` and verify exit code 0 with no failures. (Command-line execution)

---

## Technical Notes

**Root Cause:** The `power_by_squaring` function in `base/intfuncs.jl` handles negative exponents by checking `isone(-x)` to identify when `x == -1`. For unsigned integers, negation wraps around: `-typemax(UInt8)` equals `0x01`, causing `isone(-typemax(UInt8))` to return `true`. This misidentifies `typemax` as `-1`.

**Fix Scope:**
- Modify the negative-exponent path in `power_by_squaring` in `base/intfuncs.jl` to reject unsigned `typemax` values before the `isone(-x)` check
- Add regression tests to `test/math.jl`
- **Out of scope:** Changes to `src/`, `Compiler/`, `deps/`, or any behavior for signed integers, `Bool`, `BigInt`, or floating-point exponentiation

**Constraint:** The fix must preserve the existing behavior where `checked_pow(-1, n)` for any negative integer `n` returns the mathematically correct result (`-1` for odd `n`, `1` for even `n`).

**File Paths:** The canonical Julia repository structure places base library code under `base/` and tests under `test/`. If the working repository uses a different layout, the implementing agent must locate the equivalent of `base/intfuncs.jl` (the file containing `power_by_squaring` for `checked_pow`) and `test/math.jl` (the existing math test suite) within the actual repository structure and apply changes there.
