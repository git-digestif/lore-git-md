# The Git Project -- Weekly Digest for 2026/09/28 -- 2026/10/04

## The period in brief

This week (2026/09/28--2026/10/04) was exceptionally active, with **seven high-volume days** and traffic that was both **heavy and consequential**. The Git project released **v2.56.0**, marking a major milestone with **748 non-merge commits** from 104 contributors, including foundational work for Git 3.0. Three developments stand out: the **Rust-based SHA-1 backend** offering 3× performance gains, the **`git stash` autostash bugfix** addressing a long-standing NULL-dereference risk, and the **`uploadpack.lazyFetchTrusted`** series providing server-side security controls for lazy fetching. The week also saw **architectural debates** about Git’s future, including proposals for **hostname-based `includeIf`**, **shared stash functionality**, and **incremental rebase workflows**.

---

## Key developments

### Git v2.56.0 released
Junio C Hamano announced the release of **Git v2.56.0**, featuring **748 non-merge commits** from 104 contributors. This release delivers **performance optimizations** in ref handling, pack-objects, and index scanning; **new commands** (`git history drop`, `git refs`, `git replay --linearize`); and **architectural shifts** including ODB abstraction, `the_repository` removal, and Rust support enabled by default. Security hardening addresses corrupt trees, NULL-dereferences, and Windows symlink auto-detection. UX improvements include better error messages, advice for diverged branches, and `git add --resolved`. The release sets the stage for **Git 3.0**, expected in spring 2027, with foundational work on ODB pluggability and Rust integration now complete.

### Rust-based SHA-1 backend (3× performance gain)
Johannes Schindelin introduced a **four-patch series** adding an optional Rust-based SHA-1 backend (`sha1dc` crate) that delivers **~3× speedups** for `git index-pack` on Windows, WSL, and macOS. The series includes FFI integration, a runtime escape hatch (`core.sha1dcBackend=c`), thread-safety shims, and thread-safe initialization. The backend is **optional** and not enabled by default, addressing concerns about build complexity and platform support (e.g., NonStop). However, the **Rust version requirement (1.87)** and **110MB SDK size increase** for Git for Windows sparked policy-level objections from brian m. carlson, who argued the version bump should be handled in a separate policy discussion. The series remains in `seen` pending resolution of this policy question.

### `git stash` autostash bugfix (NULL-dereference risk)
D. Ben Knoble’s **five-patch series** fixing a race condition in `git stash` autostash with staged index entries was **ejected from `seen`** due to a **NULL-dereference risk** in the in-core merge logic. The series replaces subprocess-based merging with `merge-ort.c` to preserve staged entries, but Junio C Hamano identified that `result.tree` could be uninitialized on catastrophic merge failure, causing SIGSEGV. The author confirmed the issue and will address it by checking `result.clean` before dereferencing. The series also surfaced an **unrelated test breakage** in `t5520-pull.sh` caused by auto-maintenance triggering `git reflog expire`, which was diagnosed and fixed by Thomas Bachem. The fix is **high-priority** for stability, as the bug affects users of `stash.index=true` during merge operations.

### `uploadpack.lazyFetchTrusted` (server-side security control)
Christian Couder’s **five-part v5 series** introducing `uploadpack.lazyFetchTrusted` reached technical completion, providing a **server-side mechanism** for operators to explicitly declare which repositories are safe to lazy-fetch from. The series replaces the earlier client-side `GIT_NO_LAZY_FETCH=fromAccepted` proposal with a protected configuration variable, addressing security concerns about untrusted repositories triggering arbitrary code execution. Junio C Hamano’s review identified a latent design issue in the existing `safe.directory` logic (silent ignoring of non-existent paths), which Christian acknowledged and proposed addressing in a follow-up patch. The series is now ready for maintainer review and integration.

### `gitmergeconflicts(7)`: New man page for conflict resolution
Julia Evans’ **RFC series** converting Git’s merge conflict advice into a dedicated `gitmergeconflicts(7)` man page saw significant progress. The series adds a **"WHAT IS A MERGE CONFLICT?"** section, improves the explanation of `diff3` conflict style, and clarifies terminology. Junio C Hamano and Patrick Steinhardt proposed **embedding commit details directly into conflict markers**, transforming them from passive indicators into active resolution tools. The discussion also gained momentum around making `merge.conflictstyle=diff3` the default, with both maintainers strongly advocating for this change. The series is now in its third iteration and nearing completion.

### MIDX reachability closure fixes
Taylor Blau’s **eight-patch v2 series** fixing corner cases in Git’s multi-pack-index (MIDX) reachability closure received substantive review feedback from Jeff King and Elijah Newren. The series addresses scenarios where MIDXs can end up containing objects not closed under reachability, violating the MIDX’s invariant and causing bitmap generation failures. The key debate centers on the **data structure choice for `extra_roots`**: Newren’s test cases demonstrated that an `oid_array` preserves critical path and namehash ordering required for delta compression quality, while an `oidset` would introduce functional regressions. The series remains in review, with the `oidset` vs. `oid_array` question unresolved.

### Unified `post-worktree` hook
Domen Kožar posted **v3 of the unified `post-worktree` hook**, consolidating three separate hooks (`post-worktree-add`, `post-worktree-remove`, `post-worktree-move`) into a single hook with a subcommand-style interface. The hook takes four arguments: the event name (`add`, `move`, or `remove`), the worktree identifier, the old absolute path, and the new absolute path. The series directly addresses the need for **reliable observability** for worktree lifecycle events, even when the caller is uncontrolled (e.g., IDEs, manual deletion). Junio C Hamano remains fundamentally opposed to hooks, but the series is technically complete and has garnered support from users like Maciej Ciemborowicz, who reported real-world use cases for IDE integration.

### `--[no-]range-diff-notes` feature series
Kristoffer Haugsbakk’s **feature series** adding `--[no-]range-diff-notes` options to `git format-patch` reached technical completion in its **v5 iteration**. The series introduces two new CLI options: `--range-diff-notes=<ref>` (requires an explicit ref argument) and `--no-range-diff-notes` (suppresses notes in the range-diff without fallback). The interaction rule is simple: if no range-diff notes options are given, the range-diff uses the same notes as the patches; otherwise, it uses only the explicitly specified notes. The series is now ready for integration, with all maintainer feedback addressed.

---

## In brief

**`git filter-branch` bugfix** -- Michele Locati and Grant Moyer fixed a commit mapping inversion regression in `git filter-branch --state-branch` introduced in Git 2.50.0. The fix preserves the documented "from_commit:to_commit" format and includes comprehensive test coverage.

**Reftable reflog timezone encoding fix** -- Josh McKinney and Patrick Steinhardt fixed a discrepancy in Git’s reftable backend where reflog timezone offsets were written as signed HHMM integers instead of signed minutes, breaking interoperability with JGit. The fix aligns Git with the reftable specification.

**`gitbreaking-changes(7)` manpage** -- Julia Evans proposed converting Git’s `BreakingChanges` document into a manpage (`gitbreaking-changes(7)`), improving discoverability via `git help` and `man`. The RFC seeks feedback on the anchor style trade-off (manual vs. auto-generated anchors).

**CI optimizations** -- Harald Nordgren and Tamir Duberstein improved GitHub Actions failure reporting for leak sanitizer jobs and capped job parallelism to CPU count. The changes reduce CI noise and improve debuggability.

**`git stash create` options** -- Kazumasa Shigeta extended `git stash create` to support `--include-untracked` and `--all` options, addressing a long-standing inconsistency with `git stash push` and `git stash save`.

**`includeIf "hostname:..."`** -- Isabella Caselli proposed adding a hostname-based `includeIf` condition for Git configuration, enabling multi-machine dotfile sharing. The RFC seeks feedback on hostname normalization and discoverability trade-offs.

**`git(1)` man page rewrite** -- Julia Evans rewrote the `git(1)` intro to orient users toward the help system, replacing outdated tutorial references. Ben Knoble praised the conciseness but questioned the omission of `git help cmd`.

**`gittutorial-2` removal** -- Julia Evans proposed removing the obsolete `gittutorial-2` document, simplifying the documentation landscape for future improvements.

**Git for Windows 2.56.0** -- Johannes Schindelin announced the release, dropping Windows 8.1 support and fixing platform-specific bugs (e.g., large-object commits, parallel checkout crashes).

**`git -C` alias bug** -- A user reported that `git -C other_repo` via alias in a worktree mis-sets `GIT_DIR` and `GIT_COMMON_DIR`, causing commands to operate on the wrong repository.

**`git branch --delete-merged` squash-merge detection** -- Harald Nordgren extended `git branch --delete-merged` to recognize branches whose changes have been squash-merged or rebase-merged into an upstream branch. Phillip Wood identified backward-compatibility issues that need addressing.

**`git config --list --show-origin` inconsistency** -- Christophe Lohr reported that the command displays `.git/config` as a relative path, creating confusion about the `.git` directory’s location.

**`git submodule` root-relative URLs** -- Stanislav Aleksandrov proposed `^/` syntax for submodule URLs, allowing them to be resolved relative to the superproject’s remote server root.

**`rerere` race condition fix** -- Thomas Bachem’s three-patch series fixes a long-standing race condition in the `rerere` subsystem, introducing `--skip-locked` mode for `git rerere gc`.

**`git restore-mtimes` proposal** -- Artem S. Tashkinov proposed a new command to preserve per-file historical modification timestamps derived from Git commit history.

**Stash sharing through remotes** -- Hanan Arshad proposed a porcelain workflow for `git stash` to publish, list, import, and remove stashes through remotes, leveraging `git stash export` and `git stash import`.

**Incremental rebase workflows** -- Alejandro Colomar and Nico Williams debated whether incremental rebase conflict resolution should be a new command (`git brebase`), a flag for `git rebase`, or the default behavior. The discussion highlights design trade-offs between simplicity and functionality.

**Windows memory regression with fsmonitor** -- Pierre Bruno reported unexpectedly high memory consumption (around 1 GB per process) when running `git status`-like commands in Git 2.56.0.windows.1 with fsmonitor enabled. The regression is under investigation.

**`git repack --dry-run --drop-filtered` regression** -- Coy Geek reported that `git repack --dry-run --drop-filtered` incorrectly mutates repository storage despite its documented promise to only list candidate objects. The regression is critical and requires a fix.

**`remote.<name>.refmap` design ambiguity** -- Junio C Hamano identified a design ambiguity in Harald Nordgren’s v6 series introducing `remote.<name>.refmap`, where both `remote.<name>.fetch` and `remote.<name>.refmap` are configured. The series remains blocked until this design point is settled.

---

## Looking ahead

The next week is likely to see continued activity around several ongoing efforts:

- **Rust-based SHA-1 backend**: The policy-level objection over Rust version requirements must be resolved before the series can proceed. Expect discussion about whether Git should align with Debian stable or adopt a more aggressive version policy.
- **`git stash` autostash bugfix**: The NULL-dereference risk must be addressed before the series can re-enter `seen`. The fix is high-priority for stability.
- **`uploadpack.lazyFetchTrusted`**: The series is technically complete and ready for maintainer review. Integration is likely if no further issues are identified.
- **MIDX reachability closure fixes**: The `oidset` vs. `oid_array` debate must be resolved before the series can graduate to `next`. Expect further discussion about the trade-offs between memory efficiency and delta compression quality.
- **`gitmergeconflicts(7)`**: The series is nearing completion, with the next iteration likely to include the proposed conflict marker enhancements and `diff3` promotion.
- **Incremental rebase workflows**: The design debate about whether to integrate incremental conflict resolution into `git rebase` or introduce a new command (`git brebase`) will continue. Expect further discussion about the trade-offs between simplicity and functionality.