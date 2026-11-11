# Course correction PER-221 (auto from critic exhaustion)

Authoritative for the next run. Human: review/edit this file, then re-run:
`adlc start PER-221 --skip-dev` or `adlc review request-changes PER-221 --note "..."` then `adlc resume PER-221`.

## Critic mismatches to resolve

- **AC count mismatch between requirements and story:** Requirements.md lists 6 acceptance criteria (AC-1 through AC-6), but story Tests section lists 8 test cases. The discrepancy: requirements AC-1 specifies testing all three unsigned types (UInt8, UInt64, UInt128) as a single criterion with "three test cases in test/math.jl", while the story splits this into three separate tests (Test 1, Test 2, Test 3). This is a structural mismatch — the story's Tests section should either consolidate tests 1-3 into one entry matching AC-1, or requirements should split AC-1 into three separate criteria.

- **File path contradiction with repo context:** Ticket, requirements, and story reference `base/intfuncs.jl` and `test/math.jl` as canonical Julia upstream paths. The repo tree shows only `src/` at root with no `base/` or `test/` directories. Story acknowledges this in Description and Technical Specification ("The provided repo tree shows only `src/` at the root. The implementing agent must locate the actual files..."), but the contradiction remains unresolved: the story still lists test commands using paths that do not exist in the working repository (`./usr/bin/julia --startup-file=no test/runtests.jl math`). Per consistency rules, file paths must match repo context or carry an explicit "see LLD" pointer when the story cannot resolve them — the story's "must locate" instruction defers resolution but does not satisfy the rule that paths must be concrete or explicitly deferred.

- **Test count verification gap:** Requirements AC-1 states "three test cases in test/math.jl" as verification for the UInt8/UInt64/UInt128 rejection. Story Test section lists Test 1, Test 2, Test 3 as separate entries all verifying AC-1. If these three tests are the "three test cases" referenced in AC-1's verification, the count is consistent (3 = 3). However, AC-6 verification states "run ... and verify exit code 0 with no failures" as a single command execution, and the story lists Test 8 as a separate test case for AC-6. The requirements verification for AC-6 describes a pipeline check (command-line execution), not a discrete test case — the story elevates this to a numbered test, creating a mismatch between what the requirements describe (a build/CI check) and what the story lists (a test case).

- **Language/stack mismatch:** Repo context states the repository is Julia with test command `julia --startup-file=no test/runtests.jl math`, but the repo tree shows only C/C++ source files under `src/` (`.c`, `.cpp`, `.S` assembly files, Makefile). No `.jl` files exist in the tree. The story and requirements assume a Julia base library layout that does not match the provided repository structure. This contradicts the consistency rule that stack/language must match repo context.

## Repair rules

- Keep ticket/repo labels (`repo:*`) and out-of-scope constraints.
- Prefer one complete story LLD per involved repo when possible; do not leave truncated mid-sentence files.
- Align AC counts, test counts, and file paths across requirements ↔ HLD ↔ breakdown ↔ stories.
- Rewrite incomplete artifacts fully (no cutoffs).
