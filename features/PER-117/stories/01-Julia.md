# PER-117: Fix `binomial(typemin(T), one(T))` to return `typemin(T)`

**Type:** story  
**ID:** 117  
**Status:** ready-for-dev  
**Assignee:** (unassigned)  
**Epic:** (none — standalone backlog item)  
**Repo:** Julia  
**Track:** quick  

---

## Description

`binomial(typemin(Int8), Int8(1))`, and the same call for `Int64` and `Int128`, currently return `0`. The identity `binomial(n, 1) == n` holds for negative integers, and `typemin(T)` is representable, so the result should be `typemin(T)`.

**Root cause:** The negative-`n` path computes `-n + k - one(T)`. Negating `typemin(T)` wraps to `typemin(T)` again (two's complement overflow), the value stays negative, and the `k > n` check returns zero before the `k == 1` case is reached.

**Fix approach:** Handle `k == 1` before the negative-`n` transform, or use widened arithmetic to prevent the overflow.

---

## Technical Specification

### Implementation Location

- **File:** One of the source files under `src/` where `binomial` is currently implemented (exact file TBD during implementation; likely `src/builtin*.c` or `src/builtins.c` based on the repo tree)
- **Test file:** `test/math.jl` (existing test suite location for math functions)

### Implementation Approach

Two valid approaches; choose one:

**Option A: Early return for `k == 1`**  
Check `k == 1` and return `n` immediately, before any arithmetic transform for negative `n`. This is the simplest fix and matches the mathematical identity directly.

```julia
# Pseudocode (adapt to actual implementation language/structure)
if k == one(T)
    return n
end
# ... existing negative-n logic follows
```

**Option B: Widened arithmetic**  
Use a wider integer type during the `-n + k - one(T)` computation for negative `n`, then cast back. This prevents the overflow but is more complex.

**Recommendation:** Option A is preferred — simpler, clearer intent, and no performance cost.

### Test Cases

Add to `test/math.jl`:

```julia
# Regression test for binomial(typemin(T), 1) returning typemin(T)
@test binomial(typemin(Int8), Int8(1)) == typemin(Int8)        # -128
@test binomial(typemin(Int64), Int64(1)) == typemin(Int64)      # -9223372036854775808
@test binomial(typemin(Int128), Int128(1)) == typemin(Int128)  # -170141183460469231731687303715884105728

# Confirm existing identity still holds
@test binomial(5, 1) == 5
@test binomial(-5, 1) == -5
@test binomial(0, 1) == 0
```

### Files Changed

1. `src/<binomial-implementation-file>` — add the `k == 1` early return (or widened arithmetic, if Option B is chosen)
2. `test/math.jl` — add the regression test cases listed above

---

## Acceptance Criteria

### AC-1: `binomial(typemin(T), one(T))` returns `typemin(T)` for signed integer types

**Given** the user calls `binomial(typemin(T), one(T))` where `T` is `Int8`, `Int64`, or `Int128`  
**When** the function executes  
**Then** it returns `typemin(T)` for that type

**Verify:**
```bash
# In Julia REPL or test harness:
binomial(typemin(Int8), Int8(1)) == typemin(Int8)        # Expected: true (-128)
binomial(typemin(Int64), Int64(1)) == typemin(Int64)      # Expected: true (-9223372036854775808)
binomial(typemin(Int128), Int128(1)) == typemin(Int128)  # Expected: true (-170141183460469231731687303715884105728)
```

---

### AC-2: Existing `binomial(n, 1)` identity preserved

**Given** the user calls `binomial(n, 1)` for any representable integer `n`  
**When** the function executes  
**Then** it returns `n`

**Verify:**
```bash
# In Julia REPL or test harness:
binomial(5, 1) == 5       # Expected: true
binomial(-5, 1) == -5     # Expected: true
binomial(0, 1) == 0       # Expected: true
```

---

### AC-3: Implementation handles `k == 1` before negative-`n` arithmetic transform

**Given** the current implementation computes `-n + k - one(T)` for negative `n`  
**When** `k == 1` and `n == typemin(T)`  
**Then** the function either:
- Returns `n` directly when `k == 1` before applying the negative-`n` transform, OR
- Uses widened arithmetic to prevent overflow during the transform

**Verify:** Code review of the implementation in `src/<binomial-implementation-file>` confirms one of these approaches is present

---

### AC-4: Regression test added to `test/math.jl`

**Given** the fix is implemented  
**When** the test suite runs  
**Then** `test/math.jl` contains explicit test cases for `binomial(typemin(T), one(T))` where `T ∈ {Int8, Int64, Int128}`

**Verify:** Inspection of `test/math.jl` shows the new `@test` lines for `typemin(Int8)`, `typemin(Int64)`, and `typemin(Int128)` with `k == 1`

---

### AC-5: Existing test suite passes

**Given** the fix is implemented  
**When** the math test suite runs  
**Then** all tests pass with no new failures

**Verify:**
```bash
./usr/bin/julia --startup-file=no test/runtests.jl math
# Expected exit status: 0
# Expected output: no test failures reported
```

---

### AC-6: Whitespace checks pass

**Given** the fix is implemented  
**When** the whitespace check runs  
**Then** the check passes with no violations

**Verify:**
```bash
./usr/bin/julia --startup-file=no contrib/check-whitespace.jl
# Expected exit status: 0
# Expected output: no whitespace violations reported
```

---

## Out of Scope

- Changes to `binomial` behavior for any `k` value other than `1`
- Changes to `binomial` behavior for unsigned integer types
- Performance optimization beyond what is necessary for correctness
- Fixing other edge cases in `binomial` not related to `k == 1` and `typemin(T)`

---

## Notes

- **Planner focus:** Focus on the `k == 1` special case fix before negative-`n` arithmetic; note the test location `test/math.jl` and the two verification commands.
- **Verification commands:** Both `./usr/bin/julia --startup-file=no test/runtests.jl math` and `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl` must pass before delivery.
- **File paths:** All paths are relative to the Julia repo root. The `binomial` implementation file will be identified during the dev step; the test file `test/math.jl` already exists per the repo tree.

---

## References

- Ticket: PER-117 (this story is derived from the original ticket)
- Repo context: `repo-context/Julia.md`
- Repo tree: `repo-tree/Julia`
