# Story — Return `(missing, missing)` from `sincospi(missing)`

**Story ID:** PER-114  
**Type:** Bug Fix  
**Epic:** N/A (Standalone fix)  
**Track:** Quick  
**Estimated Points:** 1  
**Status:** Ready for Development

---

## Problem Statement

The `sincospi(missing)` function currently returns a single `missing` value instead of a tuple `(missing, missing)`, causing destructuring assignment to fail with `MethodError: no method matching iterate(::Missing)`. This is inconsistent with `sincos(missing)` and `sincosd(missing)`, which both return `(missing, missing)`, and violates the documented API that `sincospi` returns a `(sine, cosine)` tuple.

**Root Cause:** In `base/math.jl`, `Missing` methods are generated from a shared list around line 1325 using `@eval $(f)(::Missing) = missing`. The `:sincospi` symbol is included in this list, causing it to return a single `missing` instead of the expected tuple. The `sincos` and `sincosd` functions are not in this list and have dedicated methods that return `(missing, missing)`.

---

## Acceptance Criteria

### AC-1: `sincospi(missing)` returns a tuple
**Given** a call to `sincospi` with `missing` as the argument  
**When** the function executes  
**Then** it returns `(missing, missing)` (a tuple containing two `missing` values)

**Verification:**
```julia
result = sincospi(missing)
@test result === (missing, missing)
@test result isa Tuple{Missing, Missing}
```

---

### AC-2: Destructuring assignment succeeds
**Given** the expression `s, c = sincospi(missing)`  
**When** the assignment executes  
**Then** both `s` and `c` are bound to `missing` without raising `MethodError`

**Verification:**
```julia
s, c = sincospi(missing)
@test s === missing
@test c === missing
```

---

### AC-3: Other trig functions with `missing` are unchanged
**Given** calls to `sincos(missing)`, `sincosd(missing)`, `sinpi(missing)`, `cospi(missing)`, and `tanpi(missing)`  
**When** each function executes  
**Then**:
- `sincos(missing)` returns `(missing, missing)`
- `sincosd(missing)` returns `(missing, missing)`
- `sinpi(missing)` returns `missing`
- `cospi(missing)` returns `missing`
- `tanpi(missing)` returns `missing`

**Verification:**
```julia
@test sincos(missing) === (missing, missing)
@test sincosd(missing) === (missing, missing)
@test sinpi(missing) === missing
@test cospi(missing) === missing
@test tanpi(missing) === missing
```

---

### AC-4: Numeric behavior is unchanged
**Given** calls to `sincospi(x)` where `x` is a numeric value (e.g., `0.5`, `1.0`, `2.5`)  
**When** the function executes  
**Then** the returned tuple contains the same numeric `(sine, cosine)` values as before the fix

**Verification:**
```julia
s, c = sincospi(0.5)
@test s ≈ 1.0
@test c ≈ 0.0

s, c = sincospi(1.0)
@test s ≈ 0.0
@test c ≈ -1.0
```

---

### AC-5: Math tests pass
**Given** the command `julia --startup-file=no test/runtests.jl math`  
**When** the test suite runs  
**Then** all tests pass with no failures or errors

**Verification:** Run `julia --startup-file=no test/runtests.jl math` and confirm exit code 0.

---

### AC-6: Whitespace check passes
**Given** the command `julia --startup-file=no contrib/check-whitespace.jl`  
**When** the whitespace checker runs  
**Then** it reports no violations

**Verification:** Run `julia --startup-file=no contrib/check-whitespace.jl` and confirm exit code 0.

---

## Technical Specification

### Implementation Approach

1. **Locate the shared `Missing` method generator** in `base/math.jl` (approximately line 1325):
   ```julia
   for f in (:sin, :cos, :tan, ..., :sincospi, ...)
       @eval $(f)(::Missing) = missing
   end
   ```

2. **Remove `:sincospi` from the shared generator list** so it no longer receives the single-value `missing` method.

3. **Define a dedicated method** for `sincospi(::Missing)` immediately after the loop that returns the expected tuple:
   ```julia
   sincospi(::Missing) = (missing, missing)
   ```

4. **Add a regression test** in `test/math.jl` within the existing `Missing` behavior test section:
   ```julia
   @testset "sincospi(missing) returns tuple" begin
       @test sincospi(missing) === (missing, missing)
       s, c = sincospi(missing)
       @test s === missing
       @test c === missing
   end
   ```

### Files to Modify

| File | Change |
|------|--------|
| `base/math.jl` | Remove `:sincospi` from the shared `Missing` method generator list (around line 1325); add dedicated method `sincospi(::Missing) = (missing, missing)` immediately after the loop |
| `test/math.jl` | Add regression test verifying `sincospi(missing)` returns `(missing, missing)` and that destructuring succeeds |

### Files NOT to Modify

- No changes under `src/`, `Compiler/`, or `deps/`
- No new test files (use existing `test/math.jl`)
- No changes to `Missing` methods for `sinpi`, `cospi`, `tanpi`, `sincos`, or `sincosd`

### Edge Cases and Constraints

- **Scope:** Only `base/math.jl` and `test/math.jl` are modified
- **Backward compatibility:** Numeric behavior of `sincospi(x)` for all numeric types remains unchanged
- **Consistency:** The fix aligns `sincospi(missing)` with `sincos(missing)` and `sincosd(missing)` behavior
- **Single-value trig functions:** `sinpi`, `cospi`, and `tanpi` continue returning single `missing` values as documented

### Verification Strategy

1. **Unit tests:** The new regression test in `test/math.jl` verifies AC-1 and AC-2
2. **Integration tests:** Running `julia --startup-file=no test/runtests.jl math` verifies no regressions (AC-5)
3. **Code quality:** Running `julia --startup-file=no contrib/check-whitespace.jl` ensures style compliance (AC-6)
4. **Manual verification:** Developer can interactively test `s, c = sincospi(missing)` in a REPL

---

## Open Questions

_None — the ticket provides clear scope and implementation guidance._

---

## Notes

- The fix is a **one-line removal** from the shared generator list and a **one-line addition** of the dedicated method
- The regression test is **~5 lines** added to the existing `test/math.jl` file
- **No breaking changes:** Numeric behavior is preserved; only the `missing` edge case changes
- **Documentation impact:** The existing docstring already states `sincospi` returns a tuple, so no doc changes are needed

---

## References

- **Ticket:** Return `(missing, missing)` from `sincospi(missing)`
- **Plan:** `plan.md` (v1, quick_fix loop)
- **Repo:** `Julia` (https://github.com/pavithrachandran70/julia)
- **Base branch:** `master`
