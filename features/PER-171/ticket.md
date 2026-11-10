---
key: PER-171
type: story
status: todo
blocks: []
blocked_by: []
labels: []
repos: []
summary: "exponent \_of a\_ Rational \_just below 1 returns 0"
---

exponent((2^60 - 1) // 2^60) returns 0, so 2^exponent(q) <= q is false. The value is less than 1, so the largest integer y with 2^y <= abs(q) is -1. exponent(::Rational) uses the generic Real method, which converts to Float64. Float64(q) == 1.0, so the conversion crosses the power-of-two boundary. Integer inputs were fixed in #59508. Rationals still use the float fallback.

Expected. exponent((2^60 - 1) // 2^60) == -1, and 2^exponent(q) <= q < 2^(exponent(q)+1).

Scope. Repo Julia only. Give Rational its own exponent method that does not convert to floating point. Add the regression in the existing test/math.jl. Do not change exponent for floats or integers.

Acceptance.

* exponent((2^60 - 1) // 2^60) == -1
* 2^exponent(q) <= q for that rational
* exponent(1//1) == 0 and exponent(2//1) == 1
* ./usr/bin/julia --startup-file=no test/runtests.jl math passes
* ./usr/bin/julia --startup-file=no contrib/check-whitespace.jl passes
