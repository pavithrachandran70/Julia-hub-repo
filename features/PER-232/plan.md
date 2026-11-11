# Plan — PER-232 (v1)

- made from: ticket
- source: LLM planner
- needs plan review: yes
- track hint: quick
- loop: quick_fix
- delivery: delivery
- PR fixes run automatically: pr_fix_minor, pr_fix_moderate (other PR comments go to a person)

A single-repo bug fix in Julia's rationalize function to honor the tol parameter for out-of-range integers; quick_fix loop fits because the behavior is clearly stated and the fix is localized to one function.

## Steps

1. [x] **analyst (inline_story)** — Convert the bug description into a Given/When/Then story with concrete acceptance criteria and implementation notes.
   - focus: Focus on the two paths: handling tol when a valid approximation exists within tolerance, and deciding whether to reject the keyword or use it. Include edge cases like typemin/typemax boundaries.
2. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
3. [ ] **dev** _(auto)_ — Loop delivery.
4. [ ] **pre_pr_qa** _(auto)_ — Loop delivery.
5. [ ] **open_pr** _(auto)_ — Loop delivery.
6. [ ] **pr_review** _(auto)_ — Loop delivery.
7. [ ] **merge** _(auto)_ — Loop delivery.

## Skipped roles

- **critic** — Single inline story with no other artifacts to cross-check for consistency.
