---
key: PER-232
type: story
status: todo
blocks: []
blocked_by: []
labels: []
repos: []
summary: "`rationalize(Int, x)` \_ignores\_ `tol` \_for out-of-range integer inputs"
---

Description:

The integer method of `rationalize` accepts a `tol` keyword but does not read it. When the input is larger than `typemax(T)`, it returns `1//0` straight away:

```
julia> rationalize(Int8, 128; tol=1)
1//0

julia> abs(128 - 127//1) <= 1
true
```

`127//1` is a valid `Rational{Int8}` and is within the requested tolerance, so `1//0` is wrong. Infinity is outside every finite tolerance. The related change in #61297 made `rationalize(Int8, 200f0)` return `127//1` instead of `1//0`, but this integer overload kept the old behavior.

Expected: Either honor `tol` and return a qualifying approximation, or reject the keyword instead of silently ignoring it.
