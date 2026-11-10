# Plan — PER-170 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: quick
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Single-repo bug fix for exponent(::Rational) that currently returns incorrect values due to Float64 conversion; quick_fix loop fits because the scope and implementation are fully defined in the ticket.

## Steps

1. [ ] **analyst (inline_story)** — Turn the ticket into a single implementable story with Given/When/Then acceptance criteria.
   - focus: Focus on the specific rational exponent bug with Float64 conversion; the ticket already provides clear scope (Julia repo only, math.jl tests) and technical approach (new Rational-specific exponent method).
2. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
3. [ ] **dev** _(auto)_ — Loop delivery.
4. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
5. [ ] **open_pr** _(auto)_ — Loop delivery.
6. [ ] **pr_review** _(auto)_ — Loop delivery.
7. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **critic** — Single inline story produces only one artifact (requirements.md with one story), so there are no multiple documents to cross-check for consistency.
