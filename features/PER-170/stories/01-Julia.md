# Story PER-170: Fix exponent() for Rational to avoid Float64 conversion error

## Metadata

```yaml
id: PER-170
type: story
title: Fix exponent() for Rational to avoid Float64 conversion error
epic: N/A
priority: normal
estimate: 2
status: draft
assignee: null
created: 2025-01-24
updated: 2025-01-24
```

## Description

The `exponent()` function for `Rational` values currently returns incorrect results when the rational is just below 1. For example, `exponent((2^60 - 1) // 2^60)` returns `0` instead of the correct value `-1`. This occurs because the generic `Real` method converts the rational to `Float64`, and `Float64((2^60 - 1) // 2^60)` rounds to `1.0`, crossing a power-of-two boundary.

The fix implements a Rational-specific `exponent` method that computes the result directly from the numerator and denominator without floating-point conversion, ensuring the mathematical property `2^exponent(q) <= q < 2^(exponent(q)+1)` holds for all rational values.

**Scope:** Julia repository only. Add regression test to existing `test/math.jl`. Do not modify `exponent` behavior for floats or integers.

## Acceptance Criteria

### AC1: Edge case near 1 returns correct exponent
**Given** a rational value just below 1: `(2^60 - 1) // 2^60`  
**When** `exponent()` is called on this value  
**Then** it returns `-1`

### AC2: Mathematical invariant holds for edge case
**Given** the rational `q = (2^60 - 1) // 2^60`  
**When** computing `2^exponent(q)`  
**Then** the result satisfies `2^exponent(q) <= q`

### AC3: Common rational exponents are correct
**Given** the rationals `1//1` and `2//1`  
**When** `exponent()` is called on each  
**Then** `exponent(1//1)` returns `0` and `exponent(2//1)` returns `1`

### AC4: Math tests pass
**Given** the implementation is complete  
**When** running `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all tests pass with no failures

### AC5: Whitespace check passes
**Given** the implementation is complete  
**When** running `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl`  
**Then** the check passes with no violations

## Technical Specification

### Implementation Approach

Add a new method `exponent(r::Rational)` that computes the exponent directly from the numerator and denominator without floating-point conversion:

```julia
function exponent(r::Rational)
    if iszero(r)
        throw(DomainError(r, "Cannot compute exponent of zero"))
    end
    # For rational q = num/den, exponent is:
    # floor(log2(abs(num))) - floor(log2(abs(den)))
    # which equals exponent(num) - exponent(den) when both are non-zero integers
    return exponent(numerator(r)) - exponent(denominator(r))
end
```

This leverages the existing `exponent` implementation for integers (fixed in #59508) to compute the result without any floating-point operations.

### Files to Modify

- **base/rational.jl** or **base/intfuncs.jl** (wherever `exponent` is currently defined for `Real`):
  - Add the new `exponent(::Rational)` method
  
- **test/math.jl**:
  - Add regression tests covering AC1-AC3

### Mathematical Foundation

For a rational `q = num/den` where both are non-zero integers:
- `exponent(q)` should return the largest integer `y` such that `2^y <= abs(q)`
- This equals `exponent(abs(num)) - exponent(abs(den))`
- The existing integer `exponent` returns `floor(log2(abs(n)))` for integer `n`

### Edge Cases Covered

1. Rationals just below 1 (the bug case): `(2^60 - 1) // 2^60`
2. Rational equal to 1: `1//1`
3. Rationals greater than 1: `2//1`, `3//2`
4. Negative rationals: behavior inherits from `abs()` handling

## Verification

### Manual Testing
```julia
julia> exponent((2^60 - 1) // 2^60)
-1

julia> q = (2^60 - 1) // 2^60; 2^exponent(q) <= q
true

julia> exponent(1//1)
0

julia> exponent(2//1)
1
```

### Automated Testing
- Run `./usr/bin/julia --startup-file=no test/runtests.jl math`
- Run `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl`

## References

- Related: #59508 (fixed integer `exponent` implementation)
- Repository: https://github.com/pavithrachandran70/julia
- Base branch: `master`

## Notes

- The implementation reuses the corrected integer `exponent` method from #59508
- No changes needed to float or integer `exponent` behavior
- Test coverage is added to existing test file rather than creating a new test file
