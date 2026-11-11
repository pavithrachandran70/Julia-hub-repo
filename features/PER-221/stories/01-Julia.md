# Story — checked_pow mistakes the largest unsigned integer for -1

**ID:** Story-1  
**Type:** story  
**Status:** draft  
**Epic:** N/A (standalone backlog story)  
**Assignee:** unassigned  
**Created:** 2025-01-24  
**Updated:** 2025-01-24

---

## Description

`Base.Checked.checked_pow(typemax(UInt8), -1)` incorrectly returns `0xff` instead of throwing a `DomainError`. The same issue affects `UInt64` and `UInt128`. The bug occurs because the negative-exponent path in `power_by_squaring` checks `isone(-x)` to identify when `x == -1`. For unsigned integers, negation wraps around: `-typemax(UInt8)` equals `0x01`, causing `isone(-typemax(UInt8))` to return `true` and misidentifying `typemax` as `-1`.

The fix must preserve the existing behavior where `checked_pow(-1, n)` for any signed negative integer `n` returns the mathematically correct result (`-1` for odd `n`, `1` for even `n`).

**Scope:**
- Modify the negative-exponent path in the C/C++ runtime implementation to reject unsigned `typemax` values before the `isone(-x)` check
- Add regression tests to the existing test harness
- **Out of scope:** Changes unrelated to the `power_by_squaring` negative-exponent path, or any behavior for signed integers, `Bool`, `BigInt`, or floating-point exponentiation

**Repository note:** The working repository at `/home/pavc/Documents/Julia-Repo` contains the C/C++ runtime implementation (`src/` directory). Julia standard library directories (`base/`, `test/`, `contrib/`) containing `.jl` files are not present in this repository tree; the bug fix targets the runtime layer.

---

## Acceptance Criteria

### AC-1: Reject typemax for unsigned integers with negative exponents

**Given** an unsigned integer type `T` where `T` is `UInt8`, `UInt64`, or `UInt128`  
**When** `checked_pow(typemax(T), -1)` is called  
**Then** a `DomainError` is thrown

---

### AC-2: Reject other unsigned integers with negative exponents

**Given** an unsigned integer value that is not `typemax`  
**When** `checked_pow(UInt8(2), -1)` is called  
**Then** a `DomainError` is thrown

---

### AC-3: Preserve positive exponent behavior for unsigned integers

**Given** an unsigned integer with a positive exponent  
**When** `checked_pow(UInt8(2), 2)` is called  
**Then** the result is `0x04`

---

### AC-4: Preserve signed -1 behavior with negative exponents (odd)

**Given** a signed integer value of `-1` and an odd negative exponent  
**When** `checked_pow(Int8(-1), -1)` is called  
**Then** the result is `Int8(-1)`

---

### AC-5: Preserve signed -1 behavior with negative exponents (even)

**Given** a signed integer value of `-1` and an even negative exponent  
**When** `checked_pow(Int8(-1), -2)` is called  
**Then** the result is `Int8(1)`

---

### AC-6: Math test suite passes

**Given** the fix is implemented  
**When** the math test suite is executed with `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all tests pass

---

## Tests

### Test 1: Reject typemax(UInt8) with negative exponent
**Verifies:** AC-1  
**Approach:** Call `checked_pow(typemax(UInt8), -1)` and verify `DomainError` is thrown  
**Location:** See LLD

### Test 2: Reject typemax(UInt64) with negative exponent
**Verifies:** AC-1  
**Approach:** Call `checked_pow(typemax(UInt64), -1)` and verify `DomainError` is thrown  
**Location:** See LLD

### Test 3: Reject typemax(UInt128) with negative exponent
**Verifies:** AC-1  
**Approach:** Call `checked_pow(typemax(UInt128), -1)` and verify `DomainError` is thrown  
**Location:** See LLD

### Test 4: Reject non-typemax unsigned with negative exponent
**Verifies:** AC-2  
**Approach:** Call `checked_pow(UInt8(2), -1)` and verify `DomainError` is thrown  
**Location:** See LLD

### Test 5: Preserve positive exponent for unsigned
**Verifies:** AC-3  
**Approach:** Call `checked_pow(UInt8(2), 2)` and verify result equals `0x04`  
**Location:** See LLD

### Test 6: Preserve signed -1 with odd negative exponent
**Verifies:** AC-4  
**Approach:** Call `checked_pow(Int8(-1), -1)` and verify result equals `Int8(-1)`  
**Location:** See LLD

### Test 7: Preserve signed -1 with even negative exponent
**Verifies:** AC-5  
**Approach:** Call `checked_pow(Int8(-1), -2)` and verify result equals `Int8(1)`  
**Location:** See LLD

### Test 8: Math test suite passes
**Verifies:** AC-6  
**Approach:** Run `./usr/bin/julia --startup-file=no test/runtests.jl math` and verify exit code 0  
**Location:** Command-line execution

---

## Technical Specification

### Root Cause Analysis

The C/C++ runtime implementation of the `power_by_squaring` function handles negative exponents by checking `isone(-x)` to identify when `x == -1`. This works correctly for signed integers where `-(-1) == 1`. However, for unsigned integers, the negation operator wraps around:

- `-typemax(UInt8)` wraps to `0x01` (since `0xff + 0x01 = 0x00` in modular arithmetic)
- `isone(0x01)` returns `true`
- The function incorrectly treats `typemax(UInt8)` as `-1` and returns it instead of throwing `DomainError`

### Implementation Plan

**Files to investigate and modify (within `src/`):**

The implementing agent must locate the C/C++ runtime implementation of `power_by_squaring` or its equivalent checked arithmetic function within the `src/` directory tree. Candidate files based on naming patterns in the repo-tree include:

- `src/builtins.c` — may contain built-in arithmetic operations
- `src/codegen.cpp` — may contain power/exponentiation lowering
- Files matching `*arith*`, `*check*`, or `*pow*` patterns if present

**Changes required:**

1. **Add unsigned typemax guard in negative-exponent path:**
   - Locate the negative exponent branch in the power-by-squaring implementation (the `if p < 0` or equivalent block)
   - Before the existing `isone(-x)` check, add a condition to reject unsigned integers where `x == typemax(typeof(x))`
   - Throw the appropriate exception/error with a message indicating negative exponents are not supported for unsigned typemax values
   - The guard must only affect unsigned integer types (`UInt8`, `UInt16`, `UInt32`, `UInt64`, `UInt128`)

2. **Preserve signed -1 handling:**
   - The existing `isone(-x)` check must continue to work for signed integers
   - Use type checking to distinguish between signed and unsigned paths

**Test location:**

The implementing agent must locate or create appropriate test cases within the existing test infrastructure. The ticket specifies running `./usr/bin/julia --startup-file=no test/runtests.jl math`, which suggests tests live outside the `src/` directory in a Julia standard library test suite not present in this repo-tree. The implementing agent must determine how to add regression tests for this C/C++ runtime bug fix.

### Constraints

- **Preserve all existing signed integer behavior:** Signed `-1` must continue to work correctly with negative exponents
- **No impact on `Bool`, `BigInt`, or floating-point:** These types use different code paths and must remain unchanged
- **No changes under `Compiler/` or `deps/`:** The bug is in the C/C++ runtime implementation under `src/` only
- **Repository layout:** The current repo-tree at `/home/pavc/Documents/Julia-Repo` shows only the C/C++ runtime (`src/` directory). The implementing agent must work within this constraint and determine the correct C/C++ file(s) to modify.

---

## Dependencies

**Blocks:** None  
**Blocked by:** None  
**After:** None

---

## Notes

- **Repository structure clarification:** The repo-tree shows only `src/` (C/C++ runtime) at `/home/pavc/Documents/Julia-Repo`. This story targets the C/C++ runtime layer, not Julia standard library `.jl` files. The implementing agent must locate the appropriate C/C++ implementation files within `src/` that handle `checked_pow` or `power_by_squaring`.
- The regression tests should be added to the existing test harness used by the `julia --startup-file=no test/runtests.jl math` command
- The fix should include a comment explaining why the unsigned typemax guard is necessary to prevent future removal
- The ticket specifies whitespace checks must pass (`julia --startup-file=no contrib/check-whitespace.jl`); this is enforced by the existing CI pipeline and does not require an additional acceptance criterion
- Consider whether other functions using `isone(-x)` have the same vulnerability (though this story's scope is limited to `checked_pow` only)
- **Open question:** The exact C/C++ file(s) containing the `power_by_squaring` or `checked_pow` runtime implementation must be identified by the implementing agent through code inspection of the `src/` directory.

---

## References

- Ticket: checked_pow mistakes the largest unsigned integer for -1
- Implementation scope: C/C++ runtime files under `src/` in `/home/pavc/Documents/Julia-Repo`
- Test command: `./usr/bin/julia --startup-file=no test/runtests.jl math`
- Lint command: `julia --startup-file=no contrib/check-whitespace.jl`
- Repository context: Julia source at `https://github.com/pavithrachandran70/julia`, base branch `master`
