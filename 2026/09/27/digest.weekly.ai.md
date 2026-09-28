# Git Mailing List Digest: 2026/09/21 -- 2026/09/27

## The period in brief
This week (2026/09/21--2026/09/27) saw **high-volume, technically dense traffic** across Git's core subsystems. The list was unusually active for a non-release week, with **six days of substantive discussion** and no quiet periods. Two long-running regression fixes -- the `reference-transaction` hook and `git stash` autostash -- reached critical milestones, while architectural debates flared around the "safe" `strbuf` API and Git 3.0's version numbering. **Do not miss**: the `reference-transaction` hook v6 pivot, the `git stash` autostash bugfix completion, and the Git 3.0 timeline clarification.

---

## Key developments

### Reference-transaction hook regression fix pivots to transaction-layer design (v6)
The six-iteration series fixing a **Git 2.31 regression** where the `reference-transaction` hook received all-zero OIDs on branch/tag deletion reached a **major architectural pivot** in v6. Earlier versions modified `refs_delete_refs()` to preserve old OIDs, but Junio C Hamano and Patrick Steinhardt identified a **critical design flaw**: the patch turned deletions into an all-or-nothing operation, breaking `git fetch --prune` and `git remote prune` where users expect unconditional deletion even if some refs are concurrently updated.

The v6 series abandons the `refs_delete_refs()` approach entirely and instead **moves the fix into the transaction layer itself**. The new design records a separate "observed old value" for updates where callers didn't supply one, using this value only as hook input without setting `REF_HAVE_OLD` or constraining the update. This ensures the hook always receives meaningful old OIDs while preserving the performance gains of the original optimization. The series includes updated regression tests and is rebased onto `master`.

**Key participants**: Maciej Ciemborowicz (author), Junio C Hamano, Patrick Steinhardt, Karthik Nayak.
**Current status**: v6 posted; addresses all prior architectural concerns. Ready for final review before `next`.

---

### `git stash` autostash bugfix completes five-patch series
A **critical race condition** in `git stash` autostash handling of staged index entries when `stash.index=true` was fixed in a **five-patch series** that reached completion. The bug, which could corrupt staged entries during merge operations, was caused by a non-reentrant filesystem operation in `save_autostash()`. The fix replaces subprocess-based index merging (`git diff-tree` and `git apply`) with in-core merging via `merge-ort.c`, eliminating the SIGSEGV risk.

The series is now **technically complete**, incorporating all review feedback:
- Fixed a **tree argument order bug** in `merge_incore_nonrecursive()` (flagged by Junio)
- Resolved a **memory leak** (missing `merge_finalize()` call)
- Expanded test coverage to verify both worktree and index state on failure
- Structured as: cleanup (already merged), refactoring, test additions, and the functional fix

**Key participants**: D. Ben Knoble (author), Junio C Hamano, Phillip Wood.
**Current status**: Patches 2--5 finalized; ready for integration. The series touches `builtin/stash.c`, `merge-ort.c`, and the test suite.

---

### Git 3.0 timeline clarified; Rust portability advances
The **Git 3.0 timeline discussion** saw significant clarification this week. Junio C Hamano proposed a **tentative three-phase timeline** (Git 2.98 in December 2026, Git 2.99 in March 2027, Git 3.0 in April 2027) but emphasized its flexibility, noting the stabilization period for Git 2.99 may require additional maintenance releases. The discussion highlighted two key developments:
1. **Rust portability**: Ramsay Jones reported successful Rust compilation on Cygwin using an unofficial toolchain, providing the first public data point on Rust's viability outside mainstream platforms. While encouraging, the toolchain's experimental status limits immediate practicality.
2. **JGit readiness**: The JGit project's pluggable backend architecture is now production-ready, as validated by Google's internal Git servers and GerritForge's global refdb backend. This removes a major ecosystem blocker for Git 3.0's SHA-256 interoperability goals.

**Key participants**: Junio C Hamano, Johannes Schindelin, Ramsay Jones, brian m. carlson.
**Current status**: Timeline remains provisional; Rust support is a goal but not guaranteed for Git 3.0 due to portability concerns (e.g., NonStop platform).

---

### "Safe" `strbuf` API RFC sparks architectural debate
Derrick Stolee's RFC series introducing a **"safe" `strbuf` API subset** that cannot call `die()` received **substantive feedback** from Junio C Hamano and Jeff King, sparking a broader architectural debate. The series aims to harden core service routines as part of Git's libification effort, but reviewers raised two key concerns:
1. **API design**: Junio argued the goal should be to harden the *entire* `strbuf` API rather than carve out a niche "safe" subset, and suggested renaming higher-level utilities (e.g., `strbuf_realpath()`) to avoid the `strbuf_` prefix.
2. **Implementation bugs**: Two bugs were identified:
   - A **memory leak** in `srealloc()` (where `realloc()` failure clobbers the caller's pointer)
   - A **misleading error message** in `safe_memory_limit_check()` (reporting the original `git_alloc_limit` value instead of the effective limit)

Jeff King proposed an alternative design using **fixed-size, stack-allocated buffers** to avoid `malloc()` entirely, arguing this would better serve the trace2 use case. The discussion remains open, with no consensus on the path forward.

**Key participants**: Derrick Stolee (author), Junio C Hamano, Jeff King, Phillip Wood.
**Current status**: RFC stage; architectural direction unresolved. The series touches `strbuf.c`, `strbuf.h`, and test infrastructure.

---

### `gitmergeconflicts(7)`: New man page centralizes conflict resolution guidance
Julia Evans posted a **seven-patch series** introducing `gitmergeconflicts(7)`, a new man page consolidating merge conflict resolution guidance previously scattered across `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull`. The new guide is organized as a **practical, example-driven document** with sections covering:
- When conflicts occur
- Resolution paths
- Merge conflict markers
- Tools for handling conflicts
- Edge cases (e.g., `git rebase` inverts "ours" and "theirs")

The series updates existing command man pages to link to the new guide and adds it to the build system. Junio requested mechanical adjustments (e.g., updating `Documentation/meson.build` and folding the `.gitattributes` addition into the first patch), but the core content is uncontroversial.

**Key participants**: Julia Evans (author), Junio C Hamano, Jeff King.
**Current status**: Merged to `seen`; ready for integration after mechanical fixes.

---

### `git repack` concurrent-push data-loss fix advances
A **six-patch series** fixing a race condition where concurrent pushes can cause `git repack -d` to delete packs whose objects were never copied elsewhere reached a critical milestone. Junio C Hamano **acked patch 2/5 (v3)** after the author moved the kept-pack cache invalidation logic to `packfile.c`, addressing a layering concern. The series is now **ready for `next`**, with the remaining patches under final review.

The fix is **critical for repository integrity** and has been validated in GitLab CI. The series touches `builtin/repack.c`, `packfile.c`, and `object-file.c`.

**Key participants**: Patrick Steinhardt (author), Junio C Hamano.
**Current status**: Patch 2/5 acked; remaining patches under review.

---

### CI and build system improvements land
Several **CI and build system improvements** landed this week, addressing long-standing pain points:
1. **Windows unit tests**: Karthik Nayak fixed a regression where unit tests were being skipped on Windows CI due to a mismatch in test-slice indexing.
2. **Meson build optimizations**: Patrick Steinhardt posted a **seven-patch series** reducing clean build times by ~35% (from ~6.8s to ~5.0s) by:
   - Avoiding redundant compilation of HTTP sources
   - Using precompiled headers for test helpers and unit tests
   - Fixing outdated completion helpers
3. **Leak sanitizer reporting**: Harald Nordgren improved leak sanitizer failure reporting in GitHub Actions by stopping tests at the first failure and emitting detailed annotations.

**Key participants**: Karthik Nayak, Patrick Steinhardt, Harald Nordgren.
**Current status**: All patches merged or ready for integration.

---

## In brief

**`reference-transaction` hook omits new ref on branch rename** -- The v2 patch confines copy/rename state to per-ref-update data, preserving transaction genericity and enabling multi-ref operations. Ready for review.

**`git-p4` shell injection fix (v2)** -- Anupam Mediratta posted v2 of the patch fixing a shell injection vulnerability in `git-p4`, restoring the original error-handling behavior while preserving the shell injection mitigation. Ready for integration.

**`git reflog expire` default expiry fix (v5)** -- The v5 patch restores the correct default expiry periods for `git reflog expire` (reachable=90d, unreachable=30d), addressing a Git 2.50 regression. Approved by Junio and ready for `next`.

**`git stash` autostash verbosity regression** -- Junio C Hamano identified a verbosity regression in the final patch of the `git stash` autostash bugfix series, where the `.verbosity` field was unconditionally set to `0` during the index merge and never restored. A minimal fix has been proposed.

**`git var` extension series (v9)** -- Andrew Pleeter posted v9 of the series implementing a unified, scriptable interface for Git's identity and signing configuration. The controversial `GIT_SIGNING_KEY` variable was removed due to ambiguity across signing mechanisms. Ready for integration.

**`git ls-files` untracked cache reuse (v2)** -- The v2 series expanded to three patches, enabling cache sharing between `git status -unormal` and `git status -uall` and teaching `git ls-files` to reuse the cache. Benchmarks show a **7.3× speedup** (241 ms → 33 ms) for wildcard queries on a synthetic tree with 100 k files.

**`git repo info` path-related keys (v7)** -- K Jayatheerth’s series adding eight new path-related keys to `git repo info` reached v7, marking the series as mechanically complete and correctness-fixed. Ready for substantive review.

**`git format-patch` `--[no-]range-diff-notes` (v2)** -- Kristoffer Haugsbakk posted v2 of the series, addressing Junio C Hamano’s usability concerns by eliminating the toggle mechanism and enforcing explicit separation between patch notes and range-diff notes.

**`git status --ignored` pathspec substring matching bugfix (v2)** -- René Scharfe posted v2 of the bugfix, incorporating Junio’s squash-in fix for a NULL-dereference crash in the new `dir_match()` helper. Ready for integration.

**`git shortlog` regression fix** -- Jeff King’s two-patch series fixing the `(null)` regression in `git shortlog` error messages is under maintainer approval and a strong candidate for inclusion in `next`.

**`git-interpret-trailers` documentation** -- Kristoffer Haugsbakk improved clarity of `trailer.<key-alias>.cmd` examples in the `git-interpret-trailers` man page.

**Localization updates** -- Jiang Xin submitted l10n updates for Git 2.56.0, adding Afrikaans and Brazilian Portuguese translations and refreshing eight existing languages.

---

## Looking ahead
The next week is likely to see **integration of several high-priority series**, including:
- The `reference-transaction` hook v6 series (now architecturally sound)
- The `git stash` autostash bugfix (verbosity regression fix pending)
- The `git repack` concurrent-push data-loss fix (patch 2/5 already acked)
- The `gitmergeconflicts(7)` man page (mechanical fixes pending)

**Ongoing architectural debates** to watch:
- The "safe" `strbuf` API RFC (direction unresolved)
- Git 3.0 version numbering (default changes still under discussion)
- The `git var` extension series (v9 ready for integration)

**Late-posted series** likely to see review:
- K Jayatheerth’s `git repo info` path-related keys (v7)
- Kristoffer Haugsbakk’s `--[no-]range-diff-notes` (v2)