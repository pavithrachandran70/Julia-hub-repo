# Plan — PER-221 (v2)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: standard
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Single-repo bug fix in Julia's checked_pow for unsigned integers: the negative-exponent path misidentifies typemax(UInt) as -1 due to wrap-around. Loop quick_fix fits because scope, acceptance criteria, and the one-file change are all stated in the ticket.

## Steps

1. [ ] **analyst (inline_story)** — Convert the ticket into a single implementable story with Given/When/Then AC and tech notes for the fix.
   - focus: Focus on the unsigned wrap-around logic in power_by_squaring's negative-exponent path in base/intfuncs.jl. The fix must preserve signed -1 behavior and only affect unsigned typemax rejection.
2. [ ] **critic (consistency_check)** — Cross-check that the story's acceptance criteria match the ticket and that the fix scope excludes signed integers, Bool, BigInt, floats, and all paths outside base/intfuncs.jl and test/math.jl.
3. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
4. [ ] **dev** _(auto)_ — Loop delivery.
5. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
6. [ ] **open_pr** _(auto)_ — Loop delivery.
7. [ ] **pr_review** _(auto)_ — Loop delivery.
8. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — Scope and acceptance criteria are concrete and fully specified in the ticket; no product sign-off needed for an internal math bug.
- **designer** — The ticket already identifies the file, function, and root cause (unsigned wrap in isone(-x) check); no low-level design step needed.
- **test_architect** — The ticket lists exact test cases and the existing test file; no separate test design artifact needed.
