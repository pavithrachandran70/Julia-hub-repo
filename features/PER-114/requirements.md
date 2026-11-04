# Requirements — Return `(missing, missing)` from `sincospi(missing)`

**Story ID:** PER-114  
**Type:** Bug Fix  
**Epic:** N/A (Standalone fix)  
**Estimated Points:** 1

---

## Problem Statement

The `sincospi(missing)` function currently returns a single `missing` value instead of a tuple `(missing, missing)`, causing destructuring assignment to fail with `MethodError: no method matching iterate(::Missing)`. This is inconsistent with `sincos(missing)` and `sincosd(missing)`, which both return `(missing, missing)`, and violates the documented API that `sincospi` returns a `(sine, cosine)` tuple.

---

## Acceptance Criteria

### AC-1: `sincospi(missing)` returns a tuple
**Given** a call to `sincospi` with `missing` as the argument  
**When** the function executes  
**Then** it returns `(missing, missing)` (a tuple containing two `missing` values)

### AC-2: Destructuring assignment succeeds
**Given** the expression `s, c = sincospi(missing)`  
**When** the assignment executes  
**Then** both `s` and `c` are bound to `missing` without raising `MethodError`

### AC-3: Other trig functions with `missing` are unchanged
**Given** calls to `sincos(missing)`, `sincosd(missing)`, `sinpi(missing)`, `cospi(missing)`, and `tanpi(missing)`  
**When** each function executes  
**Then**:
- `sincos(missing)` returns `(missing, missing)`
- `sincosd(missing)` returns `(missing, missing)`
- `sinpi(missing)` returns `missing`
- `cospi(missing)` returns `missing`
- `tanpi(missing)` returns `missing`

### AC-4: Numeric behavior is unchanged
**Given** calls to `sincospi(x)` where `x` is a numeric value (e.g., `0.5`, `1.0`, `2.5`)  
**When** the function executes  
**Then** the returned tuple contains the same numeric `(sine, cosine)` values as before the fix

### AC-5: Math tests pass
**Given** the command `julia --startup-file=no test/runtests.jl math`  
**When** the test suite runs  
**Then** all tests pass with no failures or errors

### AC-6: Whitespace check passes
**Given** the command `julia --startup-file=no contrib/check-whitespace.jl`  
**When** the whitespace checker runs  
**Then** it reports no violations

---

## Technical Notes

### Implementation Approach

1. **Locate the shared `Missing` method generator** in `base/math.jl` (approximately line 1325) that currently generates `@eval $(f)(::Missing) = missing` for a list of functions including `:sincospi`.

2. **Remove `:sincospi` from the shared generator list** so it no longer receives the single-value `missing` method.

3. **Define a dedicated method** for `sincospi(::Missing)` that returns `(missing, missing)`, similar to how `sincos` and `sincosd` handle `Missing`.

4. **Add a regression test** in `test/math.jl` that:
   - Verifies `sincospi(missing)` returns `(missing, missing)`
   - Verifies destructuring `s, c = sincospi(missing)` succeeds
   - Confirms the other trig functions' `missing` behavior is unchanged (optional sanity check)

### Files to Modify

- `base/math.jl` — Remove `:sincospi` from shared list; add dedicated `sincospi(::Missing) = (missing, missing)` method
- `test/math.jl` — Add regression test for the new behavior

### Files NOT to Modify

- No changes under `src/`, `Compiler/`, or `deps/`
- No new test files (use existing `test/math.jl`)
- No changes to `Missing` methods for `sinpi`, `cospi`, `tanpi`, `sincos`, or `sincosd`

### Verification Strategy

Run the math test suite and whitespace checker as specified in AC-5 and AC-6. Manual verification of destructuring assignment behavior is covered by the new regression test.

---

## Open Questions

_None — the ticket provides clear scope and implementation guidance._

---

## References

- Ticket: Return `(missing, missing)` from `sincospi(missing)`
- Plan: `plan.md` (v1, quick_fix loop)
