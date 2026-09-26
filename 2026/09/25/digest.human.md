# Git mailing list daily digest for 2026/09/25

## The day in brief
The Git project saw significant progress on several fronts today. The `http.sslVerifyStatus` security feature's test infrastructure issues were resolved by switching from GnuTLS to OpenSSL. A critical bug in `git stash` autostash handling was fixed, with all technical concerns addressed. The `git replay` command gained GPG signing support, and a new `gitmergeconflicts(7)` man page was proposed to centralize conflict resolution guidance. Several CI improvements were made, including better leak sanitizer reporting and dynamic parallelism adjustments.

## Notable threads

### `http.sslVerifyStatus` test infrastructure resolved
**What changed**: The test failure in the `http.sslVerifyStatus` security feature was resolved by switching from the GnuTLS backend to OpenSSL for libcurl.

**Problem/goal**: Git's HTTPS certificate verification was enhanced with OCSP staple validation, but tests were failing on systems using GnuTLS due to its incomplete OCSP stapling support.

**Subsystem**: HTTP transport, security

**Impact**: This environmental fix removes the last blocker for the feature, which was previously reverted from `next` due to test infrastructure issues. The core functionality remains intact in `master`, and the feature is now ready to graduate.

### Key details

- The test failure was caused by using GnuTLS backend for libcurl, which lacks full OCSP stapling support
- Switching to OpenSSL backend resolves the issue
- The fix is purely environmental and doesn't affect the core feature implementation

### `git stash` autostash bugfix completed
**What changed**: A critical bug in `git stash` autostash handling of staged index entries was fixed, with all technical concerns now addressed.

**Problem/goal**: The `--force-if-includes` safety mechanism in `git stash` contained a race condition that could corrupt staged index entries during autostash operations when `stash.index=true` was set.

**Subsystem**: Stash, merge operations

**Impact**: This bugfix replaces subprocess-based index merging with in-core merging via `merge-ort.c`, eliminating the race condition. The series is now technically complete and ready for integration.

### Key details

- The fix addresses a critical correctness issue in the tree argument order for `merge_incore_nonrecursive()`
- All memory leaks and test robustness issues have been resolved
- The series consists of four patches: cleanup, refactoring, test additions, and the functional fix
- The broader argument-order standardization between merge functions is acknowledged as a future cleanup task

### `git replay` gains GPG signing support
**What changed**: The `git replay` command now supports GPG commit signing via a new `-S` option.

**Problem/goal**: The `git replay` command, introduced in Git 2.46, lacked commit signing support, which was inconsistent with other Git commands.

**Subsystem**: Replay, commit signing

**Impact**: This feature addition aligns `git replay` with other Git commands that support signing, while maintaining plumbing command consistency by ignoring `commit.gpgSign`.

### Key details

- Two-patch series: bugfix for error handling and feature implementation
- New CLI options: `-S`/`--gpg-sign` and `--no-gpg-sign`
- Comprehensive test coverage for signing behavior and failure modes
- The series explicitly ignores `commit.gpgSign` to maintain plumbing command consistency

### Git version numbering and 3.0 planning
**What changed**: The discussion about Git 3.0 version numbering and scope continued, with new information about JGit's production readiness.

**Problem/goal**: The Git project is planning its next major release (3.0) and needs to decide on version numbering, scope, and timeline.

**Subsystem**: Project planning, ecosystem coordination

**Impact**: The discussion affects downstream projects and the broader Git ecosystem, particularly regarding breaking changes and foundational work like SHA-256 support.

### Key details

- JGit's pluggable backend architecture is production-ready, as validated by Google's internal Git servers and GerritForge's global refdb backend
- Git's experimental Rust support compiles successfully on Cygwin using an unofficial toolchain
- The stabilization period for Git 2.99 may require additional maintenance releases before Git 3.0 is finalized
- Security process improvements are being implemented, with GitLab increasing staffing for report triage

### New `gitmergeconflicts(7)` man page proposed
**What changed**: A new `gitmergeconflicts(7)` man page was proposed to centralize merge conflict resolution guidance.

**Problem/goal**: Git's current documentation scatters merge-conflict advice across multiple command man pages, leading to duplication and user confusion.

**Subsystem**: Documentation

**Impact**: This new guide aims to improve user experience by providing a single, comprehensive resource for conflict resolution, while reducing maintenance burden.

### Key details

- The new guide consolidates information from `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull`
- It provides practical examples and clarifies terminology like "ours" vs. "theirs"
- The series must balance centralization with preserving command-specific nuances
- Cherry-pick-specific details may need to be retained in the cherry-pick man page alongside the cross-reference

## In brief
- **[`--force-if-includes` reflog walk]**: Tyler Cipriani confirmed that `date=0` is the only safe fallback value for reflog traversal, ensuring safety at negligible performance cost.
- **[CI improvements]**: Tamir Duberstein provided benchmark data showing a 2% speedup with dynamic parallelism in CI jobs, opting for a conservative `1x CPU count` baseline.
- **[Line-log subsystem]**: Kristofer Karlsson updated the line-log patch to trim trailing blank lines from function ranges, aligning with `git grep -W` behavior.
- **[Leak sanitizer reporting]**: Harald Nordgren improved leak sanitizer failure reporting in GitHub Actions by stopping tests at the first failure and emitting detailed annotations.
- **[Temporary pack files]**: Royce Remer registered temporary pack files with Git's tempfile subsystem to ensure cleanup on process exit, preventing disk space exhaustion.
- **[`git history` bug]**: Nikita Bobko reported that `git history fixup` updates branch refs locked during interactive rebase, causing Git to refuse further ref modifications.
- **[`git fetch-pack` design]**: Junio C Hamano clarified that `git fetch-pack` should remain a low-level command without DWIM heuristics, preserving the ability to send `want-ref` requests for arbitrary ref names.
- **[`git shortlog` regression]**: Junio C Hamano confirmed the fix for a regression where unknown options were incorrectly reported as `(null)` instead of their actual name.