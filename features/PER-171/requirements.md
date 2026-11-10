# Requirements — Rational Exponent Boundary Fix

**Ticket:** PER-171  
**Feature:** Fix exponent(::Rational) power-of-two boundary crossing

---

## Given (Preconditions)

**GIVEN** the Julia codebase is on the `master` branch

**AND** `exponent(::Rational)` currently uses the generic `Real` fallback which converts to `Float64`

**AND** the conversion `Float64((2^60 - 1) // 2^60)` rounds to `1.0`, crossing the power-of-two boundary

**AND** integer `exponent` was fixed in PR #59508 but `Rational` still uses the float fallback

**AND** test coverage lives in `test/math.jl`

---

## When (Actions)

**WHEN** a new `exponent(::Rational)` method is implemented

**AND** the method computes the exponent using only integer arithmetic on the numerator and denominator

**AND** the method does not convert to floating point

**AND** regression tests are added to `test/math.jl` covering:
- The boundary case `exponent((2^60 - 1) // 2^60)`
- The invariant `2^exponent(q) <= q < 2^(exponent(q)+1)`
- Edge cases `exponent(1//1)` and `exponent(2//1)`

**AND** no changes are made to `exponent` methods for floats or integers

---

## Then (Outcomes)

**THEN** `exponent((2^60 - 1) // 2^60)` returns `-1` (not `0`)

**AND** the invariant `2^exponent(q) <= q` holds for `q = (2^60 - 1) // 2^60`

**AND** `exponent(1//1)` returns `0`

**AND** `exponent(2//1)` returns `1`

**AND** running `./usr/bin/julia --startup-file=no test/runtests.jl math` passes with no failures

**AND** running `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl` passes with no whitespace violations

**AND** the fix applies only to `Rational` inputs; behavior for `Float` and `Integer` inputs remains unchanged

---

## Scope Constraints

- **In scope:** New `exponent(::Rational)` method in appropriate source file, integer-only logic, regression tests in `test/math.jl`
- **Out of scope:** Changes to `exponent(::AbstractFloat)`, `exponent(::Integer)`, or any other numeric types; framework, language, ORM, or cloud infrastructure decisions

---

## Verification

Each acceptance criterion maps to a test case:

1. **Test:** `@test exponent((2^60 - 1) // 2^60) == -1`
2. **Test:** `@test 2^exponent((2^60 - 1) // 2^60) <= (2^60 - 1) // 2^60`
3. **Test:** `@test exponent(1//1) == 0`
4. **Test:** `@test exponent(2//1) == 1`
5. **CI:** `./usr/bin/julia --startup-file=no test/runtests.jl math` exits 0
6. **CI:** `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl` exits 0
