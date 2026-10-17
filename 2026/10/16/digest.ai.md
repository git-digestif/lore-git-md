# Git mailing list daily digest for 2026/10/16

## The day in brief
The Git mailing list saw active discussion on several performance optimizations, bug fixes, and documentation improvements. Key developments include a proposed fix for a severe `git log --follow` performance regression, a resolution to the long-standing branch name validation inconsistency, and multiple CI/build system updates. Several patches were marked ready for integration, while others sparked architectural debates.

## Notable threads

### Inconsistent branch name validation in porcelain commands
**What changed?**
The thread concluded with Junio C Hamano's firm stance against allowing `@` as a branch name, dismissing convenience arguments and workarounds. The maintainer's position is now unambiguous: bare `@` violates Git's ref name rules and will not be supported.

**Why it matters**
This decision resolves a usability trap where users could create but not check out branches named `@`. The inconsistency between `git check-ref-format` and porcelain commands will be fixed by rejecting `@` outright.

**Technical details**
- Commands affected: `git checkout -b`, `git switch -c`, `git branch`
- Validation discrepancy: `git check-ref-format --branch` incorrectly accepted `@`
- Subsystem: ref name validation in `refs.c` and `check-ref-format.c`

---

### [PATCH] rerere: fix race condition between rebase and background maintenance
**What changed?**
The v8 series was queued in `next` for integration testing after addressing all prior feedback. The series introduces a configurable timeout (`rerere.lockTimeout`) and a `--skip-locked` mode for `git rerere gc`.

**Why it matters**
This fixes a long-standing race condition where concurrent `rerere` operations could collide, particularly exacerbated by the geometric maintenance strategy introduced in Git 2.54.

**Technical details**
- Files touched: `rerere.c`, `builtin/am.c`, `builtin/gc.c`, `builtin/merge.c`, `builtin/rerere.c`, `builtin/stash.c`, `sequencer.c`
- New behavior: configurable timeout (default 1000ms) and silent skip for held locks
- Test coverage: `t/t4200-rerere.sh` and `t/t7900-maintenance.sh`

---

### [PATCH 0/3] imap-send: OpenSSL compatibility and correctness fixes
**What changed?**
The thread remains blocked due to a security regression in patch 2/2. Junio C Hamano identified a correctness issue where the patch's optimization might break rename-following in merge commits.

**Why it matters**
The regression affects `git log --follow` performance in repositories with complex merge structures, slowing operations from seconds to minutes.

**Technical details**
- Files touched: `log-tree.c`
- Bottleneck function: `propagate_follow_pathspec_to_parent()`
- Performance impact: 6m43s → 29s (almost matching pre-regression time of 28s)

---

### [PATCH 0/2] Add incremental connectivity check for fetch/push
**What changed?**
Kristofer Karlsson presented new benchmark data showing a 1,000-commit cutoff effectively balances performance and safety. The author agreed to avoid new `die()` calls in library code.

**Why it matters**
This opt-in feature reduces connectivity check costs from repository size to incoming change size, with benchmarks showing 2.6-2.8x speedups for large repositories.

**Technical details**
- New config keys: `transfer.connectivityCheck=full|incremental|auto`
- New internal flag: `--verify-trees-incremental`
- Performance: up to 190x faster for small pushes to large repos

---

### [PATCH v7] repo: add revision filtering support to 'git repo structure'
**What changed?**
The v7 patch was marked ready for merging after addressing the final nit. Patrick Steinhardt's sign-off ("this version looks good to me") effectively removes the last obstacle.

**Why it matters**
This feature improves diagnostic capabilities for large repositories by allowing targeted analysis of object graph subsets.

**Technical details**
- Files touched: `builtin/repo.c`, `Documentation/git-repo.adoc`, `t/t1901-repo-structure.sh`
- New CLI syntax: `git repo structure <revision>... [--not <revision>...]`
- Test coverage: 197 lines validating edge cases

---

### doc: rewrite explanation of detached HEAD state
**What changed?**
The v4 patch was merged to `master` after Junio C Hamano accepted it as "OK". A follow-up bugfix addressed test breakage caused by backtick-quoted commands.

**Why it matters**
The rewrite improves clarity for new users by explaining why detached HEAD is useful and how to recover from it.

**Technical details**
- Files touched: `Documentation/git-checkout.adoc`, `Documentation/detached-head.adoc`, `advice.c`
- New guide: `gitdetached-head(7)`
- Test coverage: `t2020-checkout-detach.sh`

## In brief
- **[PATCH v3] cache-tree: avoid crash when index is locked**: Queued in `next` after addressing all feedback
- **[PATCH 0/8] ci: housekeeping updates for CI configurations**: Marked for `next`; Junio proposed addressing Rust CI coverage
- **[PATCH] git(1): rewrite intro to orient users toward help system**: Merged to `master`
- **[PATCH v4] doc: rewrite explanation of detached HEAD state**: Merged with follow-up bugfix
- **[PATCH] files-backend: fix loose ref cache inconsistency with root refs**: Under review; architectural debate continues
- **[PATCH 0/5] Coverity-inspired integer-overflow fixes**: Under review; Phillip Wood proposed `intmax_t` arithmetic for portability
- **[PATCH 0/5] odb: fix race condition in .keep lockfile cleanup**: v2 posted addressing all feedback
- **[PATCH] t2403: use test_path_is_dir in place of test -d**: Outreachy test modernization patch
- **[PATCH] sigchain: repair signal handler chain when empty but previously pushed**: New bugfix for signal handling race
- **[PATCH] sideband: fix infinite loop in control-character config parser**: Ready for integration; Junio dismissed style nit
- **[PATCH] git(1): rewrite intro to orient users toward help system**: Merged to `master`
- **[PATCH] http: fix credential stripping regression in libcurl 8.23.0+**: New regression fix for HTTP transport
- **What's cooking in git.git (Oct 2026, #06)**: 16 new topics under review; `mc/refs-rename-transaction` breaks `seen`