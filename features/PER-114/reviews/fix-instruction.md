Fix these forge PR review comments (auto-route):

### Julia comment 4469481700
Instead of writing a comment "this is the same", let's continue to use the same loop, but branch here

```suggestion
@eval $(f)(::Missing) = $(f === :sincospi ? :(missing, missing) : (:missing))
```
