# Git mailing list daily digest for 2026/10/16

## The day in brief
The Git mailing list saw active discussion around several performance regressions, with fixes proposed for `git log --follow` and deepening fetches. Documentation improvements for detached HEAD and merge conflicts were merged, while CI housekeeping patches addressed GitLab and GitHub workflows. A long-standing debate about branch name validation (`@`) continued, and Outreachy contributions focused on test modernization and global state reduction.

## Notable threads

### Inconsistent branch name validation in porcelain commands
**What changed?**
The thread debated whether Git should allow creating branches named `@`, which is currently accepted by some commands (`checkout -b`, `switch -c`, `branch`) but rejected by `git check-ref-format`. Junio C Hamano firmly opposed preserving `@` as a branch name, calling its use a "bad habit" that violates Git's ref name rules.

### Why it matters

The inconsistency creates a usability trap: branches named `@` can be created but cannot be checked out via porcelain commands (e.g., `git switch @` fails, interpreting `@` as `HEAD`). This affects users who rely on `@` as a shorthand in complex expressions.

### Today's developments

- [D. Ben Knoble] defended shorthands as practical for interactive workflows, noting that grep results for `@` likely include legitimate reflog syntax uses (e.g., `@{upstream}`).
- [Kristoffer Haugsbakk] shared a workaround (`git symbolic-ref refs/H HEAD`) to sidestep the usability trap.
- Junio reiterated his opposition to `@` as a branch name, dismissing workarounds as insufficient.

**Subsystem**: ref name validation (`refs.c`, `check-ref-format.c`)
**Impact**: User experience; potential breaking change for workflows using `@` as a branch name.

---

### Fix race condition in .keep lockfile cleanup
**What changed?**
Justin Tobler posted v2 of a series fixing a race condition where concurrent `git-receive-pack` processes could delete each other's `.keep` lockfiles, leaving packfiles vulnerable to housekeeping deletion.

### Why it matters

The bug affects repository integrity during concurrent fetches/pushes, particularly in CI environments. The fix moves lockfile creation to the transaction commit phase, ensuring lockfiles are only written after the transaction is guaranteed to succeed.

### Today's developments

- [Justin Tobler] addressed Patrick Steinhardt's feedback: split the SHA-1 assumption fix into a separate commit, improved error messages with `errno` context, and consolidated error paths.
- The v2 series (4 patches) is now technically complete and ready for final polish.

**Files touched**: `object-file.c`, `pack-write.c`, `pack.h`, `t/t5547-push-quarantine.sh`
**Subsystem**: ODB transactions
**Impact**: Correctness; eliminates a race condition in concurrent operations.

---

### Performance regression in `git log --follow`
**What changed?**
Miklos Vajna proposed a fix for a performance regression in `git log --stat --follow` (bisected to commit 304813ed33), which slowed the command from 33 seconds to nearly 8 minutes in repositories with complex merge structures.

### Why it matters

The regression affects users tracing file history in large repositories (e.g., the Linux kernel). The fix optimizes `propagate_follow_pathspec_to_parent()` to skip redundant diffs while preserving rename-following behavior.

### Today's developments

- [Junio C Hamano] identified a potential correctness issue: the patch's `has_same` flag may incorrectly skip rename detection in merge commits where a file is semantically renamed in one parent but syntactically unchanged in the merge.
- The discussion shifted from performance to correctness, with Junio's scenario becoming a focal point for further review.

**Files touched**: `log-tree.c`
**Subsystem**: revision-walking / rename detection
**Impact**: Performance and correctness; affects `git log --follow` usability in large repositories.

---

### CI housekeeping updates
**What changed?**
Patrick Steinhardt and Junio C Hamano posted patches to modernize Git's CI configurations:
1. Fixed GitLab CI "check-whitespace" and "check-style" job failures in merge requests with >50 commits by forcing full clones (`GIT_DEPTH: 0`).
2. Updated GitHub Actions macOS runners from `macos-14` to `macos-15` after a deprecation notice.

### Why it matters

These changes ensure CI jobs remain operational and maintain coverage for Git's test suite across platforms.

### Today's developments

- [Junio C Hamano] questioned the value of GitLab's "check-whitespace" and "check-style" jobs, suggesting they might encourage poorly formatted patches.
- The macOS update patch is untested but expected to proceed without issues.

**Files touched**: `.github/workflows/main.yml`, `.gitlab-ci.yml`
**Subsystem**: CI/build infrastructure
**Impact**: CI reliability; no user-facing changes.

---

### Documentation improvements
**What changed?**
Two documentation patches were merged:
1. Julia Evans's rewrite of the detached HEAD guide (`gitdetached-head(7)`), moving it to a standalone man page and improving clarity.
2. Julia Evans's update to the `git(1)` man page, replacing outdated tutorial references with direct help commands (e.g., `git help --guides`).

### Why it matters

These changes improve discoverability and usability for new users by adopting an "every page is page one" model.

### Today's developments

- [Junio C Hamano] queued both patches for `master`.
- A follow-up fix was provided for a test breakage in the detached HEAD patch, replacing backtick-quoted commands with single quotes in `advice.c`.

**Files touched**: `Documentation/git.adoc`, `Documentation/detached-head.adoc`, `advice.c`, `t/t2020-checkout-detach.sh`
**Subsystem**: Documentation
**Impact**: User experience; no behavior changes.

---

### Outreachy contributions
**What changed?**
Several Outreachy applicants submitted patches:
1. Srujith Poojary modernized test idioms in `t/t2030-unresolve-info.sh` (replacing `test -f` with `test_path_is_file`).
2. Jane Irungu introduced herself and sought guidance on reducing Git's global state.
3. Shashank S replaced `test -d` with `test_path_is_dir` in `t/t2403-worktree-move.sh`.

### Why it matters

These contributions improve test diagnostics and introduce newcomers to Git's development workflow.

### Today's developments

- [Usman Akinyemi] and [Harald Nordgren] provided feedback on test modernization patches, suggesting scope expansions.
- Jane Irungu received procedural advice (CC mentors) but no technical guidance yet.

**Files touched**: `t/t2030-unresolve-info.sh`, `t/t2403-worktree-move.sh`
**Subsystem**: Test suite
**Impact**: Test infrastructure; no behavior changes.

## In brief
- **`git fetch --depth` regression**: [Rong Bao] argued for propagating `--depth` to submodules, aligning with `--tags` behavior.
- **`git var` refactoring**: [Junio C Hamano] requested a v2 to address cosmetic issues (sign-off placement, indentation).
- **`git stash` completion**: [Patrick Steinhardt] raised a design question about preserving option autocompletion for the deprecated `save` subcommand.
- **`git log --follow` optimization**: Junio's correctness concern may require revisiting the patch's classification logic.
- **`git describe` i18n**: [Antoni Borowski] proposed removing gettext wrappers from debug output for consistency with `transport-helper.c`.
- **`git push` HTTP regression**: [Dan] reported a libcurl 8.23.0+ issue where credentials are stripped from `CURLINFO_EFFECTIVE_URL`, causing false redirect detection.
- **`What's cooking` report**: Junio listed 16 new topics under review, including performance fixes and refactoring efforts.