# Plan — PER-187 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: quick
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Fix exponent() for Rational numbers to avoid float conversion and return correct power-of-two exponent; quick_fix loop fits because this is a single behavioral fix in one repo with clear acceptance criteria.

## Steps

1. [ ] **analyst (inline_story)** — Convert ticket into implementable Given/When/Then story with technical notes for the exponent fix.
   - focus: Focus on the algorithm for computing exponent without float conversion: use numerator and denominator exponents directly. Include edge cases from acceptance criteria.
2. [ ] **critic (consistency_check)** — Single artifact (one story) with clear acceptance criteria needs consistency check to ensure AC coverage and technical correctness.
   - focus: Verify the story's implementation approach handles the power-of-two boundary correctly and all five acceptance criteria are testable.
3. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
4. [ ] **dev** _(auto)_ — Loop delivery.
5. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
6. [ ] **open_pr** _(auto)_ — Loop delivery.
7. [ ] **pr_review** _(auto)_ — Loop delivery.
8. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **tpm_gate** — Scope and acceptance criteria are clearly stated in the ticket; no product sign-off needed for this math bugfix.
- **designer** — The fix is self-contained in one method implementation; no file plan or interface design needed.
- **test_architect** — Test cases are explicitly enumerated in acceptance criteria; no additional test design required.
