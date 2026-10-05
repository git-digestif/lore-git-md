# The Git Project -- Weekly Digest for 2026/09/28 -- 2026/10/04

## The period in brief

This week (2026/09/28 -- 2026/10/04) was exceptionally active, with **six high-volume days** of development. The period saw the **release of Git v2.56.0**, a **major performance breakthrough** with a Rust-based SHA-1 backend, and **critical bugfixes** for `git stash` autostash and reftable reflog timezone encoding. Architectural discussions about Git's future included proposals for **hostname-based `includeIf`**, **shared stash functionality**, and **optional per-repository hook consent**. The week also brought **documentation improvements**, **CI optimizations**, and **ongoing refactoring** of Git's core subsystems.

Three developments stand out: the **Rust-based SHA-1 backend** (3× performance gain), the **`git stash` autostash bugfix** (critical stability fix), and the **`uploadpack.lazyFetchTrusted`** series (security improvement for partial clones).

---

## Key developments

### Git v2.56.0 released
Junio C Hamano announced the release of Git v2.56.0, featuring **748 non-merge commits** from 104 contributors. This release marks a significant milestone in Git's evolution, with key highlights including **performance optimizations** (ref handling, pack-objects, index scanning), **new commands** (`git history drop`, `git refs`, `git replay --linearize`), and **architectural shifts** (ODB abstraction, `the_repository` removal, Rust support enabled by default). The release also includes **security hardening** (fixes for corrupt trees, NULL-dereferences, Windows symlink auto-detection) and **UX improvements** (better error messages, advice for diverged branches, `git add --resolved`).

The release sets the stage for Git 3.0, with **ODB pluggability** and **Rust integration** now enabled by default. The performance gains in `git index-pack` and ref handling address real-world scalability challenges, while new commands like `git history` and `git refs` streamline workflows.

---

### Rust-based SHA-1 backend (3× performance gain)
Johannes Schindelin introduced a **four-patch series** adding an optional Rust-based SHA-1 backend (`sha1dc` crate) that delivers **~3× speedups** for `git index-pack` on Windows, WSL, and macOS. The series includes **FFI integration**, a **runtime escape hatch** (`core.sha1dcBackend=c`), **thread-safety shims**, and **thread-safe initialization**. The Rust backend is optional and not enabled by default, addressing concerns about build complexity and platform support (e.g., NonStop).

This is a **landmark step** in Git's Rustification effort, demonstrating Rust's potential for **performance-critical paths** while maintaining backward compatibility. The **Rust version requirement (1.87)** and **110MB SDK size increase** for Git for Windows may spark debate, but the performance gains are undeniable. The series is well-tested and includes platform-specific optimizations with fallback paths.

---

### `git stash` autostash bugfix (NULL-dereference risk)
D. Ben Knoble's **five-patch series** fixing a race condition in `git stash` autostash with staged index entries was **ejected from `seen`** due to two critical issues: a **NULL-dereference risk** in the in-core merge logic and a **bisectability regression** in the test suite. The series replaces subprocess-based merging with `merge-ort.c` to preserve staged entries, but the **NULL-dereference risk** (uninitialized `result.tree` on catastrophic merge failure) could cause SIGSEGV. The author confirmed the issue and will address it by checking `result.clean` before dereferencing `result.tree`.

The bug affects users of `stash.index=true` during merge operations, causing incorrect index state preservation. The fix is **high-priority** for stability, but the **NULL-dereference risk** must be resolved before integration. The series also surfaced an **unrelated test breakage** in `t5520-pull.sh` caused by auto-maintenance triggering `git reflog expire`, which was diagnosed and fixed by Thomas Bachem.

---

### `uploadpack.lazyFetchTrusted` (v5)
Christian Couder posted v5 of the `uploadpack.lazyFetchTrusted` series, addressing all feedback from Junio's v4 review. The series replaces the earlier client-side `GIT_NO_LAZY_FETCH=fromAccepted` proposal with a **server-side protected configuration variable**, providing a mechanism for operators to explicitly declare which repositories are safe to lazy-fetch from. This addresses security concerns about untrusted repositories triggering arbitrary code execution.

The series introduces a **recursion depth counter** (limit of 5) and touches `promisor-remote.c`, `setup.c`, `upload-pack.c`, and related documentation. The v5 iteration is now **ready for maintainer review** and represents a significant security improvement for partial clones.

---

### `gitmergeconflicts(7)`: new man page for merge conflict resolution
Julia Evans' **`gitmergeconflicts(7)`** series saw significant progress this week, with new sections and design discussions. The man page consolidates scattered merge conflict advice into a dedicated guide, improving user experience and reducing documentation maintenance burden. Key additions include a **"WHAT IS A MERGE CONFLICT?"** section, improved `diff3` documentation with concrete examples, and clarification of terminology and edge cases.

Patrick Steinhardt proposed **embedding commit details directly into conflict markers**, transforming them from passive indicators into active resolution tools. The suggested format would include commit messages and dates, providing immediate context about conflicting changes. Julia endorsed the idea, highlighting the practical benefits of including commit messages and dates. The discussion also gained momentum around making `merge.conflictstyle=diff3` the default, with both Patrick and Junio C Hamano strongly advocating for this change.

---

### MIDX reachability closure fixes
Taylor Blau's **eight-patch v2 series** fixing MIDX reachability closure corner cases received substantive review feedback from Jeff King and Elijah Newren. The series addresses scenarios where MIDXs can end up containing objects not closed under reachability, violating the MIDX’s invariant and causing bitmap generation failures. Key discussions centered on the **data structure choice for `extra_roots`**, with Newren demonstrating that an `oid_array` preserves critical path and namehash ordering required for delta compression quality, while an `oidset` would introduce functional regressions.

The series ensures reachability closure for bitmaps after incremental repacks, preventing subtle corruption issues in large repositories. The unresolved `oidset` vs. `oid_array` debate could impact delta compression quality and memory usage.

---

### CI improvements: leak sanitizer and job parallelism
Two CI-related series addressed resource exhaustion in GitHub Actions:
1. **Harald Nordgren's v4 series** improves leak sanitizer failure reporting by stopping at the first leak (`--immediate`), annotating leaks with the test script name and full sanitizer output, and pointing test failures to their exact file and line. The series simplifies escaping logic, now only escaping percent signs (`%`).
2. **Tamir Duberstein's v3 series** caps job parallelism to twice the CPU count, showing a 7% speedup on Linux and avoiding a 26% slowdown on macOS compared to the one-job-per-CPU policy.

These changes **reduce CI noise** and **improve debuggability**, making it easier to identify and fix issues. The leak sanitizer improvements are particularly valuable for catching memory leaks early, while the parallelism cap prevents resource exhaustion on GitHub's 2-core Linux runners.

---

## In brief

- **`git history` signing series**: Souma's series encountered integration issues with Patrick Monette's `git replay` signing series, requiring a rebase. The `git replay` series is blocked until the `git history` work lands.
- **Reftable reflog timezone encoding fix**: Josh McKinney reported a **discrepancy** in Git's reftable backend: reflog timezone offsets are written as **signed HHMM integers**, but the **reftable specification requires signed minutes**. The mismatch breaks interoperability with JGit. Patrick Steinhardt posted a three-patch series fixing the issue.
- **`gitbreaking-changes(7)`**: Julia Evans proposed converting Git's `BreakingChanges` document into a manpage, improving discoverability via `git help` and `man`. The series includes conversion to a manpage template, replacement of message-ID references with clickable URLs, and updates to `git(1)`'s "SEE ALSO" section.
- **`git branch --delete-merged` squash-merge detection**: Harald Nordgren posted a patch extending `git branch --delete-merged` to recognize branches whose changes have been squash-merged or rebase-merged into an upstream branch. The patch addresses a common pain point for users who prefer GitHub-style squash merges.
- **`git stash` shared stashes**: Hanan Arshad proposed a **new porcelain workflow** for sharing stashes through remotes, leveraging `git stash export` and `git stash import`. The RFC seeks feedback on the **remote ref namespace** (`refs/stashes/`) and **shared stash identifiers**.
- **`includeIf "hostname:..."`**: Isabella Caselli proposed adding a **hostname-based `includeIf` condition** for Git configuration, enabling multi-machine dotfile sharing. The RFC seeks feedback on **hostname normalization** and **discoverability trade-offs**.
- **`git(1)` man page rewrite**: Julia Evans rewrote the `git(1)` intro to orient users toward the help system, replacing outdated tutorial references. Ben Knoble praised the conciseness but questioned the omission of `git help cmd`.
- **`gittutorial-2` removal**: Julia Evans proposed a **three-patch series** to remove the obsolete `gittutorial-2` document, simplifying the documentation landscape for future improvements.
- **Git for Windows 2.56.0**: Johannes Schindelin announced the release, dropping Windows 8.1 support and fixing platform-specific bugs (e.g., large-object commits, parallel checkout crashes).
- **`git -C` alias bug**: A user reported that `git -C other_repo` via alias in a worktree **mis-sets `GIT_DIR` and `GIT_COMMON_DIR`**, causing commands to operate on the wrong repository.
- **`git restore-mtimes`**: Artem S. Tashkinov proposed a new command to preserve per-file historical modification timestamps derived from Git commit history. The proposal targets repositories with independently updated blobs where mtime loss is most acute.
- **`git config --list --show-origin`**: Christophe Lohr reported that the command displays `.git/config` as a relative path, creating confusion about the `.git` directory's location.
- **`git submodule` root-relative URLs**: Stanislav Aleksandrov posted an RFC patch introducing `^/` syntax for submodule URLs, allowing them to be resolved relative to the superproject's remote server root.
- **`git remote` documentation**: Junio C Hamano suggested a **broader approach** to clarify that all `git remote` commands operate locally, rather than per-command clarifications.
- **`git stash create` options**: Phillip Wood identified backward-compatibility regressions in Kazumasa Shigeta's patch extending `git stash create` to support `--include-untracked` and `--all`.
- **`git reflog expire` regression**: Patrick Steinhardt withdrew a fast-track request for a fix after confirming the regression's **low real-world impact**. The patch restores the correct default expiry periods (reachable=90d, unreachable=30d).
- **`git backfill --dry-run`**: Derrick Stolee objected to the feature on conceptual grounds, arguing the uncompressed size estimate may not be useful.
- **`checkout -m` conflict labels**: Johannes Sixt suggested using an index extension instead of `.git/MERGE_LABELS` to store conflict labels.
- **`refs/packed-backend` optimization**: Karthik Nayak posted a performance optimization patch for the packed-refs backend.
- **`revision: add @{p} shorthand`**: Junio C Hamano queued the patch for integration, directing the author to strengthen the commit message.
- **`git filter-branch` bugfix**: Michele Locati provided a new test case for the `--state-branch` commit mapping inversion regression, ensuring the fix works correctly and prevents future regressions.
- **`parse-options` API refactoring**: Kaartic Sivaraam identified a behavioral inconsistency with `parse_options()` where the API's decision to ignore abbreviations could misinterpret arguments.
- **`rerere` race condition fix**: Thomas Bachem's three-patch series fixing a long-standing race condition in the `rerere` subsystem neared finalization. The patch introduces `--skip-locked` mode for `git rerere gc` that skips a held lock silently.
- **Windows memory regression with fsmonitor**: Pierre Bruno reported unexpectedly high memory consumption (around 1 GB per process) when running `git status`-like commands in Git 2.56.0.windows.1 with fsmonitor enabled.
- **`git repack --dry-run --drop-filtered` regression**: Coy Geek reported a critical regression where the command incorrectly mutates repository storage despite its documented promise to only list candidate objects.
- **`remote.<name>.refmap` design ambiguity**: Junio C Hamano identified a glitch and a design ambiguity in Harald Nordgren’s v6 series introducing `remote.<name>.refmap`. When both `remote.<name>.fetch` and `remote.<name>.refmap` are configured, the current code errors out with an unhelpful fatal error.
- **Zsh completion regression risk**: SZEDER Gábor identified a regression risk in a Zsh completion patch, where the current implementation would incorrectly exclude all arguments from the second word onward.
- **`post-worktree` hooks**: Domen Kožar posted v3 of the unified `post-worktree` hook, consolidating three separate hooks into one subcommand-style interface. The hook takes four arguments: the event name (`add`, `move`, or `remove`), the worktree identifier, the old absolute path, and the new absolute path.

---

## Looking ahead

Several topics are likely to dominate the next period:

- **Rust-based SHA-1 backend**: The series faces policy-level objections over the Rust version requirement (1.87). The project may need to revisit its Rust version policy before this feature can land.
- **`uploadpack.lazyFetchTrusted`**: The v5 series is ready for maintainer review and could land soon, providing a significant security improvement for partial clones.
- **`gitmergeconflicts(7)`**: The man page is nearing completion, with discussions about embedding commit details in conflict markers and making `diff3` the default conflict style.
- **MIDX reachability closure fixes**: The series is technically sound but faces ongoing debate about data structure choices (`oidset` vs. `oid_array`).
- **`git stash` autostash bugfix**: The NULL-dereference risk must be resolved before the series can land, but the fix is otherwise complete.
- **CI improvements**: The leak sanitizer series is blocked by a usability regression in GitHub Actions annotation links, while the job parallelism series is ready for broader testing.
- **Incremental rebase workflows**: The discussion about integrating bisect-driven conflict resolution into `git rebase` is ongoing, with proposals for both standalone commands and native integration.