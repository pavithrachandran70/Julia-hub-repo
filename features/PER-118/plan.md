# Plan — PER-118 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: standard
- loop: story_with_design
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Single-repo bug fix for Rational exponent function that crosses float boundary; needs inline story with implementation guidance.

## Steps

1. [x] **analyst (inline_story)** — Convert the well-defined ticket into a structured story with Given/When/Then and technical notes.
   - focus: Focus on the float conversion bug and the integer-only algorithm needed; emphasize the test locations and acceptance criteria already specified.
2. [x] **designer (lld)** — Needs concrete implementation plan for the integer-only exponent algorithm before coding.
   - focus: Design the bit-manipulation or integer-log algorithm that avoids float conversion; specify exact file locations in base/rational.jl and test/math.jl.
3. [x] **test_architect (tests)** — Five explicit acceptance criteria need to be mapped to test cases in the existing test file.
   - focus: Map each AC to a test case in test/math.jl; include the edge case (2^60-1)//2^60 and the invariant checks.
4. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
5. [ ] **dev** _(auto)_ — Loop delivery.
6. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
7. [ ] **open_pr** _(auto)_ — Loop delivery.
8. [ ] **pr_review** _(auto)_ — Loop delivery.
9. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — Scope, acceptance criteria and expected behavior are clearly stated in the ticket; no product ambiguity.
- **critic** — Single inline story with no breakdown or HLD; nothing to cross-check for consistency.
