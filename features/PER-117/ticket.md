---
key: PER-117
type: story
status: todo
blocks: []
blocked_by: []
labels: []
repos: []
summary: "`binomial(typemin(T), one(T))` \_returns zero"
---

Description:

`binomial(typemin(Int8), Int8(1))`, and the same call for `Int64` and `Int128`, return `0`. The identity `binomial(n, 1) == n` holds for negative integers, and `typemin(T)` is representable, so the result should be `typemin(T)`. The negative-`n` path computes `-n + k - one(T)`. Negating `typemin(T)` wraps, the value stays negative, and the `k > n` check returns zero before the `k == 1` case.

Expected. `binomial(typemin(T), one(T)) == typemin(T)` for `Int8`, `Int64`, and `Int128`. Other `binomial` results stay unchanged.

Scope. Repo `Julia` only. Handle `k == 1` before the negative-`n` transform, or use widened arithmetic. Add the regression in the existing `test/math.jl`.

Acceptance.

* `binomial(typemin(Int8), Int8(1)) == typemin(Int8)`, and the same for `Int64` and `Int128`
* `binomial(5, 1) == 5` and `binomial(-5, 1) == -5`
* `./usr/bin/julia --startup-file=no test/runtests.jl math` passes
* `./usr/bin/julia --startup-file=no contrib/check-whitespace.jl` passes
