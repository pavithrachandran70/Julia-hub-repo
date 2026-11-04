# Plan — PER-114 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: quick
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

Single-file behavioral fix in base/math.jl to make sincospi(missing) return a tuple like sincos and sincosd; quick_fix fits because scope, acceptance criteria, and implementation approach are stated clearly in the ticket.

## Steps

1. [x] **analyst (inline_story)** — Convert ticket into one implementable story with Given/When/Then acceptance criteria and technical notes for the single-file fix.
   - focus: Focus on the specific method signature change for sincospi(::Missing) and the regression test location; keep the scope narrow to base/math.jl and test/math.jl only.
2. [x] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
3. [ ] **dev** _(auto)_ — Loop delivery.
4. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
5. [ ] **open_pr** _(auto)_ — Loop delivery.
6. [ ] **pr_review** _(auto)_ — Loop delivery.
7. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **critic** — Single inline story produces only one artifact (requirements.md with embedded story), so there are no multiple documents to cross-check for consistency.
