# Plan — PER-171 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: quick
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Single-repo bug fix for exponent(::Rational) crossing power-of-two boundary when converting to Float64; quick_fix loop fits because the ticket provides clear acceptance criteria and the fix is self-contained in one repo.

## Steps

1. [x] **analyst (inline_story)** — Ticket already has clear acceptance criteria; inline story captures the Given/When/Then structure and technical approach.
   - focus: Focus on the power-of-two boundary issue in exponent(::Rational) and the need for an integer-only implementation that avoids Float64 conversion. Emphasize the test location (test/math.jl) and scope constraint (Rationals only, no changes to float or integer exponent methods).
2. [x] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
3. [x] **dev** _(auto)_ — Loop delivery.
4. [x] **pre_pr_qa** _(auto)_ — Loop delivery.
5. [x] **open_pr** _(auto)_ — Loop delivery.
6. [ ] **pr_review** _(auto)_ — Loop delivery.
7. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **critic** — Single inline story with no cross-artifact consistency to check.
