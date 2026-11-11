# Story PER-232 — `rationalize(Int, x)` ignores `tol` for out-of-range integer inputs

**ID:** PER-232  
**Type:** Bug Fix  
**Track:** Quick  
**Component:** `rationalize` integer method  
**File:** `src/builtins.c`

---

## Description

The integer method of `rationalize` accepts a `tol` keyword but does not read it. When the input is larger than `typemax(T)`, it returns `1//0` straight away:

```julia
julia> rationalize(Int8, 128; tol=1)
1//0

julia> abs(128 - 127//1) <= 1
true
```

`127//1` is a valid `Rational{Int8}` and is within the requested tolerance, so `1//0` is wrong. Infinity is outside every finite tolerance. The related change in #61297 made `rationalize(Int8, 200f0)` return `127//1` instead of `1//0`, but this integer overload kept the old behavior.

**Expected:** Either honor `tol` and return a qualifying approximation, or reject the keyword instead of silently ignoring it.

**Chosen Path:** Honor `tol` (Path A) — align with the float-input `rationalize` behavior from #61297 and provide maximum utility to callers.

---

## Given/When/Then Acceptance Criteria

### AC-1: Honor tolerance when a valid approximation exists above typemax

**Given** an integer input `x` that exceeds `typemax(T)` for the target rational type `Rational{T}`  
**And** a `tol` keyword argument is provided  
**And** a valid `Rational{T}` approximation exists within the specified tolerance  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns the valid approximation instead of `1//0`

**Example:**
```julia
rationalize(Int8, 128; tol=1)  # returns 127//1
# because abs(128 - 127//1) = 1 ≤ tol
```

**Verification:** Run `rationalize(Int8, 128; tol=1) == 127//1` and `rationalize(Int16, 40000; tol=10) == 32767//1`

---

### AC-2: Honor tolerance when input is below typemin

**Given** an integer input `x` that is less than `typemin(T)` for the target rational type `Rational{T}`  
**And** a `tol` keyword argument is provided  
**And** a valid `Rational{T}` approximation exists within the specified tolerance  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns the valid approximation instead of `-1//0`

**Example:**
```julia
rationalize(Int8, -129; tol=1)  # returns -128//1
# because abs(-129 - (-128//1)) = 1 ≤ tol
```

**Verification:** Run `rationalize(Int8, -129; tol=1) == -128//1` and `rationalize(Int16, -40000; tol=10) == -32768//1`

---

### AC-3: Return infinity when no valid approximation exists within tolerance

**Given** an integer input `x` that exceeds `typemax(T)` or is below `typemin(T)`  
**And** a `tol` keyword argument is provided  
**And** no valid `Rational{T}` approximation exists within the specified tolerance  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns `1//0` or `-1//0` as appropriate

**Example:**
```julia
rationalize(Int8, 200; tol=0.1)  # returns 1//0
# because abs(200 - 127//1) = 73 > 0.1
```

**Verification:** Run `rationalize(Int8, 200; tol=0.1) == 1//0` and `rationalize(Int8, -200; tol=0.1) == -1//0`

---

### AC-4: Existing behavior preserved when tol is not provided

**Given** an integer input `x` that exceeds the target type's range  
**And** no `tol` keyword argument is provided  
**When** `rationalize(T, x)` is called  
**Then** the method returns `1//0` or `-1//0` as it does currently

**Example:**
```julia
rationalize(Int8, 128)  # returns 1//0
```

**Verification:** Run `rationalize(Int8, 128) == 1//0` and `rationalize(Int8, -129) == -1//0`

---

### AC-5: Edge case at exact typemax boundary

**Given** an integer input `x` equal to `typemax(T) + 1`  
**And** a `tol` keyword argument of at least `1`  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns `typemax(T)//1`

**Example:**
```julia
rationalize(Int8, 128; tol=1)  # returns 127//1
```

**Verification:** Run `rationalize(Int8, 128; tol=1) == 127//1` and `rationalize(Int32, typemax(Int32)+1; tol=1) == typemax(Int32)//1`

---

### AC-6: Edge case at exact typemin boundary

**Given** an integer input `x` equal to `typemin(T) - 1`  
**And** a `tol` keyword argument of at least `1`  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns `typemin(T)//1`

**Example:**
```julia
rationalize(Int8, -129; tol=1)  # returns -128//1
```

**Verification:** Run `rationalize(Int8, -129; tol=1) == -128//1` and `rationalize(Int32, typemin(Int32)-1; tol=1) == typemin(Int32)//1`

---

## Technical Specification

### Location
- **File:** `src/builtins.c`
- **Method signature:** `rationalize(::Type{T}, x::Integer; tol=0) where T<:Integer`

### Implementation Approach

1. **Check if `tol` is provided:** If `tol > 0`, perform tolerance-based boundary checking.

2. **Handle positive overflow (`x > typemax(T)`):**
   - Calculate `boundary = typemax(T)//1`
   - If `abs(x - boundary) <= tol`, return `boundary`
   - Otherwise, return `1//0`

3. **Handle negative underflow (`x < typemin(T)`):**
   - Calculate `boundary = typemin(T)//1`
   - If `abs(x - boundary) <= tol`, return `boundary`
   - Otherwise, return `-1//0`

4. **Preserve existing behavior when `tol` is not provided or is zero:**
   - Return `1//0` for `x > typemax(T)`
   - Return `-1//0` for `x < typemin(T)`

### Edge Cases

- **Exact boundary match (`x == typemax(T) + 1` or `x == typemin(T) - 1`)**: With `tol >= 1`, return the boundary value.
- **Sub-unity tolerance**: When `tol < 1` and `x` is exactly one unit beyond the boundary, return infinity.
- **Multiple integer types**: Test with `Int8`, `Int16`, `Int32`, `Int64` as target types.

---

## Tests

### Test File
- **Path:** `test/runtests.jl math`

### Test Cases

```julia
# AC-1: Honor tolerance above typemax
@test rationalize(Int8, 128; tol=1) == 127//1
@test rationalize(Int16, 40000; tol=10) == 32767//1

# AC-2: Honor tolerance below typemin
@test rationalize(Int8, -129; tol=1) == -128//1
@test rationalize(Int16, -40000; tol=10) == -32768//1

# AC-3: Return infinity when out of tolerance
@test rationalize(Int8, 200; tol=0.1) == 1//0
@test rationalize(Int8, -200; tol=0.1) == -1//0

# AC-4: Preserve existing behavior without tol
@test rationalize(Int8, 128) == 1//0
@test rationalize(Int8, -129) == -1//0

# AC-5: Edge case at typemax boundary
@test rationalize(Int8, 128; tol=1) == 127//1
@test rationalize(Int32, typemax(Int32)+1; tol=1) == typemax(Int32)//1

# AC-6: Edge case at typemin boundary
@test rationalize(Int8, -129; tol=1) == -128//1
@test rationalize(Int32, typemin(Int32)-1; tol=1) == typemin(Int32)//1

# Sub-unity tolerance
@test rationalize(Int8, 128; tol=0.5) == 1//0
@test rationalize(Int8, -129; tol=0.5) == -1//0
```

---

## Notes

- This fix aligns the integer method with the float-input `rationalize` behavior established in #61297.
- The chosen approach (Path A: honor `tol`) provides maximum utility and consistency across overloads.
- All existing tests in `test/runtests.jl math` must continue to pass.
- Lint with `julia --startup-file=no contrib/check-whitespace.jl` before PR.

---

## Success Criteria

1. All six acceptance criteria pass their example cases
2. All new test cases listed under **Tests** pass
3. Existing tests in `test/runtests.jl math` continue to pass
4. Lint passes with no new whitespace issues
5. The method's behavior is consistent with the float-input `rationalize` method behavior from #61297
