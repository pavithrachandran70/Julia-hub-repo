# Requirements — PER-117

**Story:** Fix `binomial(typemin(T), one(T))` to return `typemin(T)` instead of zero

**Repo:** Julia  
**Files:** `src/` (location TBD based on current `binomial` implementation), `test/math.jl`

---

## Requirements

### REQ-1: `binomial(typemin(T), one(T))` returns `typemin(T)` for signed integer types

**Given** the user calls `binomial(typemin(T), one(T))` where `T` is `Int8`, `Int64`, or `Int128`  
**When** the function executes  
**Then** it returns `typemin(T)` for that type

**Verify:**
- `binomial(typemin(Int8), Int8(1)) == typemin(Int8)` (expected: `-128`)
- `binomial(typemin(Int64), Int64(1)) == typemin(Int64)` (expected: `-9223372036854775808`)
- `binomial(typemin(Int128), Int128(1)) == typemin(Int128)` (expected: `-170141183460469231731687303715884105728`)

---

### REQ-2: Existing `binomial(n, 1)` identity preserved

**Given** the user calls `binomial(n, 1)` for any representable integer `n`  
**When** the function executes  
**Then** it returns `n`

**Verify:**
- `binomial(5, 1) == 5`
- `binomial(-5, 1) == -5`
- `binomial(0, 1) == 0`

---

### REQ-3: Implementation handles `k == 1` before negative-`n` arithmetic transform

**Given** the current implementation computes `-n + k - one(T)` for negative `n`  
**When** `k == 1` and `n == typemin(T)`  
**Then** the function either:
- Returns `n` directly when `k == 1` before applying the negative-`n` transform, OR
- Uses widened arithmetic to prevent overflow during the transform

**Verify:** Logic inspection during code review confirms one of these approaches is implemented

---

### REQ-4: Regression test added to `test/math.jl`

**Given** the fix is implemented  
**When** the test suite runs  
**Then** `test/math.jl` contains explicit test cases for `binomial(typemin(T), one(T))` where `T ∈ {Int8, Int64, Int128}`

**Verify:** Inspection of `test/math.jl` shows the new test cases

---

### REQ-5: Existing test suite passes

**Given** the fix is implemented  
**When** `./usr/bin/julia --startup-file=no test/runtests.jl math` runs  
**Then** all tests pass with no new failures

**Verify:** Command exits with status `0` and output shows no test failures

---

### REQ-6: Whitespace checks pass

**Given** the fix is implemented  
**When** `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl` runs  
**Then** the check passes with no violations

**Verify:** Command exits with status `0` and output shows no whitespace violations

---

## Out of Scope

- Changes to `binomial` behavior for any `k` value other than `1`
- Changes to `binomial` behavior for unsigned integer types
- Performance optimization beyond what is necessary for correctness
- Fixing other edge cases in `binomial` not related to `k == 1` and `typemin(T)`

---

## Notes

- **Root cause:** The current negative-`n` path computes `-n + k - one(T)`. When `n == typemin(T)`, negation wraps to `typemin(T)` again (two's complement overflow), so the value stays negative. The `k > n` guard then returns `0` before the `k == 1` special case is reached.
- **Fix location:** The `binomial` implementation is likely in one of the files under `src/` (exact file TBD during implementation).
- **Test location:** `test/math.jl` is the existing test file for math functions; the regression test goes there.
- **Verification commands:** Both commands are specified in the ticket's acceptance criteria and must pass before delivery.
