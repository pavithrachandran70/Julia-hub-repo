---
key: PER-221
type: story
status: todo
blocks: []
blocked_by: []
labels: []
repos: []
summary: checked_pow mistakes the largest unsigned integer for -1
---

`Base.Checked.checked_pow(typemax(UInt8), -1)` returns `0xff`. The same call returns `typemax` for `UInt64` and `UInt128`. `typemax(T)` is a positive integer, so a negative exponent has no integer result. `checked_pow(UInt8(2), -1)` already throws `DomainError`. Signed `-1` is a real case: `checked_pow(-1, -1) == -1` and `(-1)^-2 == 1`.

The negative-exponent path in `power_by_squaring` (`base/intfuncs.jl`) treats `isone(-x)` as “x is -1”. Negating an unsigned integer wraps, and `-typemax(UInt8) == 0x01`, so the largest unsigned value is handled as `-1` and returned instead of rejected. `checked_pow` uses that function.

Expected. `checked_pow(typemax(T), -1)` throws `DomainError` for `UInt8`, `UInt64`, and `UInt128`. Signed `-1` to a negative power is unchanged.

Scope. Repo `Julia` only. Stop the unsigned wrap from counting as `-1` on that negative-exponent path. Add the regression in the existing `test/math.jl`. Do not change `checked_pow` for signed integers, `Bool`, or `BigInt`, and do not change float exponentiation. No changes under `src/`, `Compiler/`, or `deps/`.

Acceptance.

* `checked_pow(typemax(UInt8), -1)`, and the same call for `UInt64` and `UInt128`, throw `DomainError`
* `checked_pow(UInt8(2), -1)` still throws `DomainError`
* `checked_pow(UInt8(2), 2) == 0x04`
* `checked_pow(Int8(-1), -1) == Int8(-1)` and `checked_pow(Int8(-1), -2) == Int8(1)`
* `./usr/bin/julia --startup-file=no test/runtests.jl math` passes
* `julia --startup-file=no contrib/check-whitespace.jl` passes
