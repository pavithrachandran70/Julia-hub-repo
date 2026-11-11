# Requirements — PER-232

## Overview

**Type:** Bug Fix  
**Component:** `rationalize` integer method  
**Issue:** The `rationalize(Int, x)` method accepts a `tol` keyword argument but silently ignores it when the input exceeds the target integer type's range, returning `1//0` (infinity) instead of either honoring the tolerance or rejecting the keyword.

## Given/When/Then Acceptance Criteria

### AC-1: Honor tolerance when a valid approximation exists

**Given** an integer input `x` that exceeds `typemax(T)` for the target rational type `Rational{T}`  
**And** a `tol` keyword argument is provided  
**And** a valid `Rational{T}` approximation exists within the specified tolerance  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns the valid approximation instead of `1//0`

**Example:**
```julia
rationalize(Int8, 128; tol=1)  # should return 127//1, not 1//0
# because abs(128 - 127//1) = 1 ≤ tol
```

### AC-2: Honor tolerance when input is below typemin

**Given** an integer input `x` that is less than `typemin(T)` for the target rational type `Rational{T}`  
**And** a `tol` keyword argument is provided  
**And** a valid `Rational{T}` approximation exists within the specified tolerance  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns the valid approximation instead of `-1//0`

**Example:**
```julia
rationalize(Int8, -129; tol=1)  # should return -128//1, not -1//0
# because abs(-129 - (-128//1)) = 1 ≤ tol
```

### AC-3: Return infinity when no valid approximation exists within tolerance

**Given** an integer input `x` that exceeds `typemax(T)` or is below `typemin(T)`  
**And** a `tol` keyword argument is provided  
**And** no valid `Rational{T}` approximation exists within the specified tolerance  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns `1//0` or `-1//0` as appropriate

**Example:**
```julia
rationalize(Int8, 200; tol=0.1)  # should return 1//0
# because abs(200 - 127//1) = 73 > 0.1
```

### AC-4: Existing behavior preserved when tol is not provided

**Given** an integer input `x` that exceeds the target type's range  
**And** no `tol` keyword argument is provided  
**When** `rationalize(T, x)` is called  
**Then** the method returns `1//0` or `-1//0` as it does currently

**Example:**
```julia
rationalize(Int8, 128)  # should still return 1//0
```

### AC-5: Edge case at exact typemax boundary

**Given** an integer input `x` equal to `typemax(T) + 1`  
**And** a `tol` keyword argument of at least `1`  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns `typemax(T)//1`

**Example:**
```julia
rationalize(Int8, 128; tol=1)  # should return 127//1
```

### AC-6: Edge case at exact typemin boundary

**Given** an integer input `x` equal to `typemin(T) - 1`  
**And** a `tol` keyword argument of at least `1`  
**When** `rationalize(T, x; tol=tol)` is called  
**Then** the method returns `typemin(T)//1`

**Example:**
```julia
rationalize(Int8, -129; tol=1)  # should return -128//1
```

## Implementation Notes

### Location
- File: `src/builtins.c` or the Julia Base library file containing the `rationalize` integer method
- Method signature: `rationalize(::Type{T}, x::Integer; tol=0) where T<:Integer`

### Approach Decision
The implementation must choose one of two paths:

1. **Path A (Honor `tol`)**: When `tol` is provided, check if `typemax(T)` or `typemin(T)` (as appropriate) lies within tolerance of the input. If yes, return that boundary value as a rational; if no, return infinity.

2. **Path B (Reject `tol`)**: Throw an error or warning if `tol` is provided but the input is out of range, making the ignored parameter explicit.

**Recommended**: Path A, to align with the behavior established in #61297 for the float method and to provide maximum utility to callers.

### Test Coverage
- Test cases must cover both `typemax(T)` and `typemin(T)` boundaries
- Test with various tolerance values: exact boundary match, tolerance of 1, sub-unity tolerance where no approximation qualifies
- Test with multiple integer types: `Int8`, `Int16`, `Int32`, `Int64` as target types
- Verify existing behavior (no `tol` keyword) is unchanged

## Success Criteria

1. All six acceptance criteria pass their example cases
2. Existing tests in `test/runtests.jl math` continue to pass
3. New tests for tolerance handling at boundaries are added and pass
4. The method's behavior is consistent with the float-input `rationalize` method behavior from #61297
