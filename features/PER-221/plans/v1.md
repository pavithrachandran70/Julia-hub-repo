# Plan — PER-221 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: standard
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Single-file bugfix in base/intfuncs.jl to prevent unsigned integer wrap from being treated as -1 in checked_pow's negative-exponent path; quick_fix loop fits because scope, behavior, and acceptance criteria are fully specified.

## Steps

1. [ ] **analyst (inline_story)** — Convert ticket into implementable story with Given/When/Then AC and tech notes.
   - focus: Focus on the unsigned-wrap condition in power_by_squaring's negative-exponent path and the test cases for UInt8/UInt64/UInt128 typemax plus existing behavior preservation.
2. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
3. [ ] **dev** _(auto)_ — Loop delivery.
4. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
5. [ ] **open_pr** _(auto)_ — Loop delivery.
6. [ ] **pr_review** _(auto)_ — Loop delivery.
7. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **critic** — Single inline story produces only one artifact (requirements.md + story), so no cross-artifact consistency to check.
