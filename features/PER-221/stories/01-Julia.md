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

`Base.Checked.checked_pow(typemax(UInt8), -1)` incorrectly returns `0xff` instead of throwing a `DomainError`. The same issue affects `UInt64` and `UInt128`. The bug occurs because the negative-exponent path in `power_by_squaring` (in the Julia base library's `intfuncs.jl`) checks `isone(-x)` to identify when `x == -1`. For unsigned integers, negation wraps around: `-typemax(UInt8)` equals `0x01`, causing `isone(-typemax(UInt8))` to return `true` and misidentifying `typemax` as `-1`.

The fix must preserve the existing behavior where `checked_pow(-1, n)` for any signed negative integer `n` returns the mathematically correct result (`-1` for odd `n`, `1` for even `n`).

**Scope:**
- Modify the negative-exponent path in `power_by_squaring` in the Julia base library to reject unsigned `typemax` values before the `isone(-x)` check
- Add regression tests to the Julia math test suite
- **Out of scope:** Changes to `src/`, `Compiler/`, `deps/`, or any behavior for signed integers, `Bool`, `BigInt`, or floating-point exponentiation

**Note on file paths:** The ticket and requirements reference `base/intfuncs.jl` and `test/math.jl`, which are the canonical locations in the upstream Julia repository. The provided repo tree shows only `src/` at the root. The implementing agent must locate the actual files containing `power_by_squaring` (used by `checked_pow`) and the math test suite within the working repository's structure and apply changes there. If the repository is a fork or uses a non-standard layout, the agent must identify the equivalent files.

---

## Acceptance Criteria

### AC-1: Reject typemax for unsigned integers with negative exponents

**Given** an unsigned integer type `T` where `T` is `UInt8`, `UInt64`, or `UInt128`  
**When** `checked_pow(typemax(T), -1)` is called  
**Then** a `DomainError` is thrown

**Verification:** Call `checked_pow(typemax(UInt8), -1)`, `checked_pow(typemax(UInt64), -1)`, and `checked_pow(typemax(UInt128), -1)` and verify each throws `DomainError`.

---

### AC-2: Reject other unsigned integers with negative exponents

**Given** an unsigned integer value that is not `typemax`  
**When** `checked_pow(UInt8(2), -1)` is called  
**Then** a `DomainError` is thrown

**Verification:** Call `checked_pow(UInt8(2), -1)` and verify it throws `DomainError`.

---

### AC-3: Preserve positive exponent behavior for unsigned integers

**Given** an unsigned integer with a positive exponent  
**When** `checked_pow(UInt8(2), 2)` is called  
**Then** the result is `0x04`

**Verification:** Call `checked_pow(UInt8(2), 2)` and verify the result equals `0x04`.

---

### AC-4: Preserve signed -1 behavior with negative exponents (odd)

**Given** a signed integer value of `-1` and an odd negative exponent  
**When** `checked_pow(Int8(-1), -1)` is called  
**Then** the result is `Int8(-1)`

**Verification:** Call `checked_pow(Int8(-1), -1)` and verify the result equals `Int8(-1)`.

---

### AC-5: Preserve signed -1 behavior with negative exponents (even)

**Given** a signed integer value of `-1` and an even negative exponent  
**When** `checked_pow(Int8(-1), -2)` is called  
**Then** the result is `Int8(1)`

**Verification:** Call `checked_pow(Int8(-1), -2)` and verify the result equals `Int8(1)`.

---

### AC-6: Math test suite passes

**Given** the fix is implemented  
**When** the math test suite is executed with `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all tests pass

**Verification:** Run `./usr/bin/julia --startup-file=no test/runtests.jl math` and verify exit code 0 with no failures.

---

## Tests

### Test 1: Reject typemax(UInt8) with negative exponent
**Verifies:** AC-1  
**Approach:** Call `checked_pow(typemax(UInt8), -1)` and verify `DomainError` is thrown  
**Location:** Math test suite (see LLD)

### Test 2: Reject typemax(UInt64) with negative exponent
**Verifies:** AC-1  
**Approach:** Call `checked_pow(typemax(UInt64), -1)` and verify `DomainError` is thrown  
**Location:** Math test suite (see LLD)

### Test 3: Reject typemax(UInt128) with negative exponent
**Verifies:** AC-1  
**Approach:** Call `checked_pow(typemax(UInt128), -1)` and verify `DomainError` is thrown  
**Location:** Math test suite (see LLD)

### Test 4: Reject non-typemax unsigned with negative exponent
**Verifies:** AC-2  
**Approach:** Call `checked_pow(UInt8(2), -1)` and verify `DomainError` is thrown  
**Location:** Math test suite (see LLD)

### Test 5: Preserve positive exponent for unsigned
**Verifies:** AC-3  
**Approach:** Call `checked_pow(UInt8(2), 2)` and verify result equals `0x04`  
**Location:** Math test suite (see LLD)

### Test 6: Preserve signed -1 with odd negative exponent
**Verifies:** AC-4  
**Approach:** Call `checked_pow(Int8(-1), -1)` and verify result equals `Int8(-1)`  
**Location:** Math test suite (see LLD)

### Test 7: Preserve signed -1 with even negative exponent
**Verifies:** AC-5  
**Approach:** Call `checked_pow(Int8(-1), -2)` and verify result equals `Int8(1)`  
**Location:** Math test suite (see LLD)

### Test 8: Math test suite passes
**Verifies:** AC-6  
**Approach:** Run `./usr/bin/julia --startup-file=no test/runtests.jl math` and verify exit code 0  
**Location:** Command-line execution

---

## Technical Specification

### Root Cause Analysis

The `power_by_squaring` function in the Julia base library's integer functions module handles negative exponents by checking `isone(-x)` to identify when `x == -1`. This works correctly for signed integers where `-(-1) == 1`. However, for unsigned integers, the negation operator wraps around:

- `-typemax(UInt8)` wraps to `0x01` (since `0xff + 0x01 = 0x00` in modular arithmetic)
- `isone(0x01)` returns `true`
- The function incorrectly treats `typemax(UInt8)` as `-1` and returns it instead of throwing `DomainError`

### Implementation Plan

**Files to locate and modify:**

The implementing agent must locate the following files within the working repository structure:

1. The file containing `power_by_squaring` function used by `checked_pow` (canonically `base/intfuncs.jl` in upstream Julia)
2. The math test suite file (canonically `test/math.jl` in upstream Julia)

If the repository uses a non-standard layout, the agent must search for these files by function name or test suite organization.

**Changes required to the `power_by_squaring` implementation:**

1. **Add unsigned typemax guard in negative-exponent path:**
   - Before the existing `isone(-x)` check in the negative exponent branch of `power_by_squaring`
   - Add a condition to reject unsigned integers where `x == typemax(typeof(x))`
   - Throw `DomainError` with a message indicating negative exponents are not supported for unsigned integers (except the special case of `-1` for signed types)

2. **Preserve signed -1 handling:**
   - The existing `isone(-x)` check must continue to work for signed integers
   - The guard must only affect unsigned integer types (`UInt8`, `UInt16`, `UInt32`, `UInt64`, `UInt128`)
   - Use type checking or sign checking to distinguish between signed and unsigned paths

**Example logic structure (pseudocode):**
```julia
function power_by_squaring(x, p::Integer)
    if p < 0
        # NEW: Guard against unsigned typemax wrap-around
        if x isa Unsigned && x == typemax(typeof(x))
            throw(DomainError(x, "negative exponent not supported for unsigned typemax"))
        end
        
        # EXISTING: Handle signed -1 case
        isone(-x) ? -one(x) : throw(DomainError(x, "negative exponent"))
        # ... rest of negative exponent logic
    end
    # ... positive exponent logic
end
```

**Test cases to add to the math test suite:**

1. `@test_throws DomainError checked_pow(typemax(UInt8), -1)`
2. `@test_throws DomainError checked_pow(typemax(UInt64), -1)`
3. `@test_throws DomainError checked_pow(typemax(UInt128), -1)`
4. `@test_throws DomainError checked_pow(UInt8(2), -1)`
5. `@test checked_pow(UInt8(2), 2) == 0x04`
6. `@test checked_pow(Int8(-1), -1) == Int8(-1)`
7. `@test checked_pow(Int8(-1), -2) == Int8(1)`

### Constraints

- **Locate files within actual repository structure:** The agent must find the equivalent of `base/intfuncs.jl` and `test/math.jl` within the working repository, which may not match the canonical Julia layout
- **Preserve all existing signed integer behavior:** Signed `-1` must continue to work correctly with negative exponents
- **No impact on `Bool`, `BigInt`, or floating-point:** These types use different code paths and must remain unchanged
- **No changes under `src/`, `Compiler/`, or `deps/`:** The bug is in Julia base library code only (the ticket explicitly excludes these directories)

---

## Dependencies

**Blocks:** None  
**Blocked by:** None  
**After:** None

---

## Notes

- The regression tests should be added near existing `checked_pow` tests for discoverability
- The fix should include a comment explaining why the unsigned typemax guard is necessary to prevent future removal
- Consider whether other functions using `isone(-x)` have the same vulnerability (though this story's scope is limited to `checked_pow` only)
- The ticket specifies whitespace checks must pass (`julia --startup-file=no contrib/check-whitespace.jl`); this is enforced by the existing CI pipeline and does not require an additional acceptance criterion
- **File path resolution:** The canonical paths `base/intfuncs.jl` and `test/math.jl` may not exist at those exact locations in the working repository. The implementing agent must locate the actual files containing the `power_by_squaring` function and the math test suite, respectively, and apply changes there.

---

## References

- Ticket: checked_pow mistakes the largest unsigned integer for -1
- Implementation file: Julia base library integer functions module (locate `power_by_squaring` function)
- Test file: Julia math test suite (locate existing `checked_pow` tests)
- Test command: `./usr/bin/julia --startup-file=no test/runtests.jl math`
- Lint command: `julia --startup-file=no contrib/check-whitespace.jl`
