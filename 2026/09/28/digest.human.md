# Git mailing list daily digest for 2026/09/28

## The day in brief
The Git project saw a flurry of activity today, with key developments including the **release of Git v2.56.0**, a **new Rust-based SHA-1 backend** offering 3× performance improvements, and **critical bugfixes** for the `git stash` autostash feature and reftable reflog timezone encoding. The day also brought **architectural discussions** about Git's future, including proposals for a **hostname-based `includeIf` condition** and **shared stash functionality**, alongside **documentation improvements** and **CI optimizations**.

## Notable threads

### Git v2.56.0 released
**What changed?**
Junio C Hamano announced the release of Git v2.56.0, featuring **748 non-merge commits** from 104 contributors. Key highlights include:
- **Performance optimizations**: Ref handling, pack-objects, and index scanning improvements.
- **New commands**: `git history drop` (experimental), `git refs` toolbox, and `git replay --linearize`.
- **Architectural shifts**: ODB abstraction, `the_repository` removal, and Rust support enabled by default.
- **Security hardening**: Fixes for corrupt trees, NULL-dereferences, and Windows symlink auto-detection.
- **UX improvements**: Better error messages, advice for diverged branches, and `git add --resolved`.

### Why it matters

This release marks significant progress in Git's evolution, particularly with **ODB pluggability** and **Rust integration**, setting the stage for Git 3.0. The performance gains in `git index-pack` and ref handling address real-world scalability challenges, while new commands like `git history` and `git refs` streamline workflows.

---

### Rust-based SHA-1 backend (3× performance gain)
**What changed?**
Johannes Schindelin introduced a **four-patch series** adding an optional Rust-based SHA-1 backend (`sha1dc` crate) that delivers **~3× speedups** for `git index-pack` on Windows, WSL, and macOS. The series includes:
1. **FFI integration** of the Rust crate.
2. A **runtime escape hatch** (`core.sha1dcBackend=c`) to fall back to the C implementation.
3. **Thread-safety shims** for Windows and non-PTHREADS builds.
4. **Thread-safe initialization** of the backend.

### Why it matters

This is a **landmark step** in Git's Rustification effort, demonstrating Rust's potential for **performance-critical paths** while maintaining backward compatibility. The optional nature of the feature and runtime escape hatch address concerns about build complexity and platform support (e.g., NonStop). However, the **Rust version requirement (1.87)** and **110MB SDK size increase** for Git for Windows may spark debate.

### Key technical details

- **Files touched**: `hash.h`, `sha1dc_git.c`, `Makefile`, `Cargo.toml`, `compat/win32/pthread.*`.
- **Performance claim**: 3× speedup for `git index-pack` (benchmarks provided for Windows, WSL, macOS).
- **Build system**: `DC_SHA1_RS` flag enables the Rust backend (optional, not default).

---

### `git stash` autostash bugfix (NULL-dereference risk)
**What changed?**
D. Ben Knoble's **five-patch series** fixing a race condition in `git stash` autostash with staged index entries was **ejected from `seen`** due to two critical issues:
1. A **NULL-dereference risk** in the in-core merge logic (identified by Junio C Hamano).
2. A **bisectability regression** in the test suite (identified by Phillip Wood).

The series replaces subprocess-based merging with `merge-ort.c` to preserve staged entries, but the **NULL-dereference risk** (uninitialized `result.tree` on catastrophic merge failure) could cause SIGSEGV. The author has confirmed the issue and will address it by checking `result.clean` before dereferencing `result.tree`.

### Why it matters

The bug affects users of `stash.index=true` during merge operations, causing incorrect index state preservation. The fix is **high-priority** for stability, but the **NULL-dereference risk** must be resolved before integration. The series also surfaced an **unrelated test breakage** in `t5520-pull.sh` caused by auto-maintenance triggering `git reflog expire`, which was diagnosed and fixed by Thomas Bachem.

### Key technical details

- **Files touched**: `builtin/stash.c`, `merge-ort.c`, `t/t3903-stash.sh`.
- **Root cause**: Non-reentrant filesystem operations in `save_autostash()` corrupting index state.
- **Fix**: In-core merging via `merge-ort.c` with proper error handling.

---

### Reftable reflog timezone encoding fix
**What changed?**
Josh McKinney reported a **discrepancy** in Git's reftable backend: reflog timezone offsets are written as **signed HHMM integers** (e.g., `+05:30` → `0530`), but the **reftable specification requires signed minutes** (e.g., `+05:30` → `330`). The mismatch breaks interoperability with JGit and violates the documented format.

### Why it matters

This is a **critical interoperability issue** for the reftable backend, which is central to Git's future scalability. The fix aligns Git with the specification, ensuring compatibility with other reftable implementations (e.g., JGit). The discussion confirmed that **no migration strategy** is needed, as reflogs are short-lived (typically garbage-collected within 30 days).

### Key technical details

- **Subsystem**: Reftable backend (`refs/reftable/`).
- **On-disk format**: Timezone offset encoding (HHMM → minutes).
- **Impact**: Interoperability with JGit and other reftable consumers.

---

### `gitbreaking-changes(7)` manpage
**What changed?**
Julia Evans proposed a **four-patch RFC series** to convert Git's `BreakingChanges` document into a manpage (`gitbreaking-changes(7)`), improving discoverability via `git help` and `man`. The series includes:
1. Conversion of `BreakingChanges` to a manpage template.
2. Replacement of message-ID references with clickable URLs.
3. A note clarifying the document is a **living draft**.
4. Updates to `git(1)`'s "SEE ALSO" section.

### Why it matters

The current `BreakingChanges` document is **invisible to most users**, buried in the source tree. A manpage makes upcoming breaking changes **discoverable** and integrates them into Git's documentation ecosystem. The RFC seeks feedback on the approach, particularly the **anchor style trade-off** (manual vs. auto-generated anchors).

### Key technical details

- **Files touched**: `Documentation/BreakingChanges`, `Documentation/gitbreaking-changes.adoc`, `Documentation/git.adoc`, `command-list.txt`.
- **New manpage**: `gitbreaking-changes(7)`.

---

### CI optimizations (leak sanitizer and job parallelism)
**What changed?**
Two CI-related series addressed resource exhaustion in GitHub Actions:
1. **Harald Nordgren's v2 series** improves leak sanitizer failure reporting by:
   - Stopping at the first leak (`--immediate`).
   - Annotating leaks with the test script name and full sanitizer output.
   - Pointing test failures to their exact file and line.
2. **Tamir Duberstein's v2 series** caps job parallelism to CPU count and replaces `test_cmp` with `test_cmp_bin` in `t4205` to avoid OOM kills.

### Why it matters

These changes **reduce CI noise** and **improve debuggability**, making it easier to identify and fix issues. The leak sanitizer improvements are particularly valuable for catching memory leaks early, while the parallelism cap prevents resource exhaustion on GitHub's 2-core Linux runners.

### Key technical details

- **Files touched**: `ci/lib.sh`, `t/test-lib-github-workflow-markup.sh`, `t/test-lib.sh`, `t/t4205-log-pretty-formats.sh`.
- **New symbols**: `github_annotation_`, `finalize_test_leak_output`, `find_test_case_line_`.

---

## In brief
- **`git replay` commit signing**: Patrick Monette's series adding `-S` support to `git replay` was **blocked** by Tian Yuchen's `git history` signing series due to merge conflicts. The author confirmed the DCO requirement and addressed a **latent error-handling flaw** in `pick_regular_commit()`.
- **`git reflog expire` regression**: Patrick Steinhardt withdrew a fast-track request for a fix after confirming the regression's **low real-world impact**. The patch restores the correct default expiry periods (reachable=90d, unreachable=30d).
- **`git remote` documentation**: Junio C Hamano suggested a **broader approach** to clarify that all `git remote` commands operate locally, rather than per-command clarifications.
- **`git stash` shared stashes**: Hanan Arshad proposed a **new porcelain workflow** for sharing stashes through remotes, leveraging `git stash export` and `git stash import`. The RFC seeks feedback on the **remote ref namespace** (`refs/stashes/`) and **shared stash identifiers**.
- **`includeIf "hostname:..."`**: Isabella Caselli proposed adding a **hostname-based `includeIf` condition** for Git configuration, enabling multi-machine dotfile sharing. The RFC seeks feedback on **hostname normalization** and **discoverability trade-offs**.
- **`git(1)` man page rewrite**: Julia Evans rewrote the `git(1)` intro to orient users toward the help system, replacing outdated tutorial references. Ben Knoble praised the conciseness but questioned the omission of `git help cmd`.
- **`gittutorial-2` removal**: Julia Evans proposed a **three-patch series** to remove the obsolete `gittutorial-2` document, simplifying the documentation landscape for future improvements.
- **Git for Windows 2.56.0**: Johannes Schindelin announced the release, dropping Windows 8.1 support and fixing platform-specific bugs (e.g., large-object commits, parallel checkout crashes).
- **`git -C` alias bug**: A user reported that `git -C other_repo` via alias in a worktree **mis-sets `GIT_DIR` and `GIT_COMMON_DIR`**, causing commands to operate on the wrong repository. The bug does not manifest when run directly or from the original clone.