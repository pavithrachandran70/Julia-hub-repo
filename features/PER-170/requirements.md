# requirements.md

## Story: Fix exponent() for Rational to avoid Float64 conversion error

**ID:** PER-170  
**Type:** Bug Fix  
**Epic:** N/A  
**Priority:** Normal  
**Estimate:** 2 points

### Description

The `exponent()` function for `Rational` values currently returns incorrect results when the rational is just below 1. For example, `exponent((2^60 - 1) // 2^60)` returns `0` instead of the correct value `-1`. This occurs because the generic `Real` method converts the rational to `Float64`, and `Float64((2^60 - 1) // 2^60)` rounds to `1.0`, crossing a power-of-two boundary.

The fix implements a Rational-specific `exponent` method that computes the result directly from the numerator and denominator without floating-point conversion, ensuring the mathematical property `2^exponent(q) <= q < 2^(exponent(q)+1)` holds for all rational values.

### Acceptance Criteria

#### AC1: Edge case near 1 returns correct exponent
**Given** a rational value just below 1: `(2^60 - 1) // 2^60`  
**When** `exponent()` is called on this value  
**Then** it returns `-1`

#### AC2: Mathematical invariant holds for edge case
**Given** the rational `q = (2^60 - 1) // 2^60`  
**When** computing `2^exponent(q)`  
**Then** the result satisfies `2^exponent(q) <= q`

#### AC3: Common rational exponents are correct
**Given** the rationals `1//1` and `2//1`  
**When** `exponent()` is called on each  
**Then** `exponent(1//1)` returns `0` and `exponent(2//1)` returns `1`

#### AC4: Math tests pass
**Given** the implementation is complete  
**When** running `./usr/bin/julia --startup-file=no test/runtests.jl math`  
**Then** all tests pass with no failures

#### AC5: Whitespace check passes
**Given** the implementation is complete  
**When** running `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl`  
**Then** the check passes with no violations

### Technical Notes

- Scope is limited to the Julia repository only
- Implementation should add a new method `exponent(::Rational)` that does not convert to floating point
- The regression test should be added to the existing `test/math.jl` file
- Do not modify `exponent` behavior for floats or integers (fixed separately in #59508)

### Verification

Manual verification of the fix:
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

Automated verification via test suite as specified in AC4 and AC5.
