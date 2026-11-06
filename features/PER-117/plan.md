# Plan — PER-117 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: quick
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Fix binomial(typemin(T), one(T)) to return typemin(T) instead of zero by handling k==1 before the negative-n transform; quick_fix loop fits because it's a single-file behavioral fix in one repo with clear requirements.

## Steps

1. [ ] **analyst (inline_story)** — Convert the ticket into a single implementable story with acceptance criteria and technical notes.
   - focus: Focus on the k==1 special case fix before negative-n arithmetic; note the test location test/math.jl and the two verification commands.
2. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
3. [ ] **dev** _(auto)_ — Loop delivery.
4. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
5. [ ] **open_pr** _(auto)_ — Loop delivery.
6. [ ] **pr_review** _(auto)_ — Loop delivery.
7. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — Scope and acceptance criteria are unambiguous: fix one edge case in binomial, regression test in test/math.jl.
- **designer** — The ticket already specifies the fix approach (handle k==1 before negative-n transform or use widened arithmetic) and test location.
- **test_architect** — Test cases are already enumerated in acceptance criteria with exact expected values.
- **critic** — Only one spec document (the inline story), so no cross-artifact consistency to check.
