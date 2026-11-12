# Git mailing list daily digest for 2026/11/11

## The day in brief
The Git project made progress on several fronts today: a performance regression in `git log --follow` gained a maintainer-endorsed resolution (opt-in toggle), the `git rerere` subcommand modernization was completed, and multiple documentation improvements advanced. The day also saw substantive design discussions about worktree safety, ref-filter atoms, and ODB transactions, with several series nearing readiness for `next`.

## Notable threads

### `git log --follow` performance regression: opt-in toggle endorsed
**What changed?**
Junio C Hamano endorsed the opt-in toggle approach for `git log --follow` in non-linear history, calling it "sensible" and "far better" than reverting the original commit or accepting the performance cost as permanent. The toggle will disable `--follow` by default for merges, mitigating a ~1.6× slowdown that blocks GitLab’s deployment.

### Why it matters

The decision resolves a months-long debate about how to balance correctness (fixing lost history in `-m --follow`) with performance (a regression that made the feature unusable at scale). The toggle preserves the correctness fixes while addressing the operational impact for Git hosts.

### Key details

- **Toggle design**: Patrick Steinhardt proposed `--follow=merges` (flag with optional parameter) over a new flag (`--follow-merges`) for consistency with existing config options.
- **Default value**: Disabled by default, as agreed by Patrick, Miklos Vajna, and Junio.
- **Implementation**: Fast-tracked as a standalone patch to unblock GitLab’s deployment.
- **Series status**: The v5 correctness fixes remain ready for `next` once the toggle lands.

---

### `git rerere` subcommand modernization completed
**What changed?**
Patrick Steinhardt’s 12-patch series modernizing `git rerere`’s subcommand handling was merged to `master`. The series replaces the custom subcommand-parsing logic with Git’s standard `OPT_SUBCOMMAND()` mechanism, aligning it with other multi-subcommand built-ins like `git notes`.

### Why it matters

The refactoring is part of the broader effort to remove global state (`the_repository`) and improve code maintainability. The series also adds a `BUG()` check to catch invalid flag combinations, addressing a latent safety issue.

### Key details

- **Files touched**: `builtin/rerere.c`, `rerere.h`, `Documentation/git-rerere.adoc`, and callers in `apply.c`, `builtin/am.c`, `builtin/commit.c`, `builtin/merge.c`, `builtin/stash.c`, `rerere.c`, `sequencer.c`.
- **New symbols**: `cmd_rerere_perform()` (fallback subcommand), `cmd_rerere_clear()`, `cmd_rerere_status()`, etc.
- **Safety fix**: `BUG()` check in `rerere_cache_setup()` for invalid flag combinations (`RERERE_AUTOUPDATE` and `RERERE_NOAUTOUPDATE` both set).
- **Documentation**: Per-subcommand usage strings added to the man page.

---

### Worktree safety and ref-filter enhancements
**What changed?**
Jasper Mayone’s v5 series extending `git for-each-ref`’s `%(worktreepath)` atom and improving worktree safety gained traction. The series now uses a hashmap for O(1) lookups, restoring performance after a v4 regression, and introduces `%(currentworktree)` for conditional formatting in `git branch`.

### Why it matters

The series addresses a real safety hole: `git worktree remove` previously allowed deletion of worktrees in the middle of a rebase, bisect, or `git am` operation, risking data loss. The new atoms also improve the composability of `git branch` output.

### Key details

- **Safety fix**: `git worktree remove` now refuses to delete in-progress worktrees without `--force`.
- **Performance**: O(1) lookups for `%(worktreepath)` and `%(currentworktree)` via hashmap.
- **New atom**: `%(currentworktree)` enables conditional formatting (e.g., omitting `+` markers for rebasing branches in the current worktree).
- **Backward incompatibility**: Scripts relying on empty `%(worktreepath)` for rebasing branches or the `+` marker in `git branch` output may break.

---

### `git repo info` path-keys series design-resolved
**What changed?**
K Jayatheerth’s series adding path-related keys to `git repo info` reached design consensus after dropping contested keys (`path.hooks`, `path.cdup`) and resolving naming conventions (underscores instead of hyphens). The series is now mechanically complete and ready for merging.

### Why it matters

The series exposes filesystem locations of repository components (e.g., working tree, index) in a scriptable format, avoiding direct access to Git’s internals. The design pivot resolves all substantive objections while preserving the core functionality.

### Key details

- **Keys added**: `path.working_tree`, `path.superproject_working_tree`, `path.index`, `path.prefix`.
- **Behavior**: Keys return absolute or relative paths; empty string in bare repositories or when not in a submodule.
- **Files touched**: `builtin/repo.c`, `Documentation/git-repo.adoc`, test suite.
- **Status**: Ready for `next`; Phillip Wood confirmed all objections are resolved.

---

### `git history --verbose` output format debate
**What changed?**
The discussion about adding diff display to `git history reword` shifted to the **output format**, with Patrick Steinhardt requiring it be generalizable to all `git history` subcommands (`split`, `squash`, `fixup`). He suggested `git range-diff`’s structured format as a model, replacing the lightweight unified diff approach.

### Why it matters

The requirement forces a trade-off between immediate usability (familiar unified diff) and long-term maintainability (structured format for complex rewrites). The decision will shape how users interact with `git history`’s editor templates.

### Key details

- **Use cases**: The format must accommodate changes to commit layout, order, messages, and multiple references.
- **Proposed model**: `git range-diff`’s structured, multi-commit format.
- **Unresolved**: Whether the lightweight unified diff can coexist with the structured format.
- **Opt-in vs. opt-out**: Three reviewers (Karthik Nayak, Phillip Wood, Patrick Steinhardt) now favor default diff display with opt-out, while the author (Jelle van der Waa) still advocates for opt-in via `--verbose`.

---

### ODB transaction temporary directory creation
**What changed?**
Justin Tobler’s series to make temporary directory creation eager in ODB transactions faced design scrutiny. Karthik Nayak questioned the motivation for switching from lazy to eager creation, while Patrick Steinhardt endorsed the direction as a step toward explicit transaction management.

### Why it matters

The series addresses a correctness gap where some writes bypass the temporary directory, risking inconsistencies if the transaction fails. The debate centers on whether the performance trade-off (7% slowdown in micro-benchmarks) is justified.

### Key details

- **Behavior change**: Temporary directory created at transaction start, not on first write.
- **Performance**: 7% slowdown for `git add` writing 10,000 loose objects (atypical workload).
- **Open questions**:
  - Why are large numbers of loose objects not expected in ODB transactions?
  - Does `fsync(fd)` guarantee durability of previous writes to the same file descriptor?
- **Status**: Under review; Karthik’s design-level question remains unresolved.

---

### Documentation improvements
**`git diff` EXAMPLES rewrite**
- **What changed?**: Junio signaled readiness to queue Julia Evans’s rewrite of the `git diff` man page’s EXAMPLES section, pending resolution of the presentation format dispute (horizontal definition lists vs. tables).
- **Why it matters**: The rewrite modernizes outdated syntax (e.g., `...` → `--merge-base`) and improves readability, but Junio objects to the format due to cut-and-paste issues.

### Detached HEAD explanation

- **What changed?**: Junio suggested a phrasing tweak to clarify that `HEAD` *stores* a reference (branch or commit) rather than *being stored as* one.
- **Why it matters**: The v5 patch is otherwise ready for merging, with no further revisions expected.

---

## In brief
- **`git mktree --batch` test coverage**: Junio queued Roger Koranteng’s patch adding test coverage for `git mktree --batch`, addressing a memory leak that went unnoticed due to lack of tests.
- **`git describe` regression fix**: Oswald Buddenhagen added historical context to the `git describe` regression, noting that GitLab’s `--keep-committers` behavior likely arose from treating committer and committer-date as a single atomic unit.
- **APFS/mmap metadata quirk**: Johannes Sixt broadened the scope of the APFS/mmap issue, noting that Git’s mmap emulation on Windows exhibits the same delayed timestamp update behavior due to `FlushViewOfFile` limitations.
- **`git repack --drop-filtered`**: kongfankaikai proposed that `git repack --drop-filtered` should skip blobs referenced by the index and continue processing, optionally warning about skipped blobs.
- **`git rerere` safety check**: Patrick Steinhardt noted a latent safety issue in `enum rerere_flags` where both `RERERE_AUTOUPDATE` and `RERERE_NOAUTOUPDATE` could be set simultaneously, though no caller intentionally does so.
- **Test fix for `t2401-worktree-prune.sh`**: Junio acked Anders Kaseorg’s fix for a brittle `sed` substitution that could corrupt paths if the trash directory contained `w2`.
- **ODB abstraction**: Patrick Steinhardt and Kaartic Sivaraam reviewed a series moving the loose-object backend into its own source file, with minor feedback on documentation and header hygiene.
- **Config segfault fixes**: Thomas Bachem’s 15-patch series fixing segfaults when config keys lack values gained maintainer endorsement, with Junio citing historical precedent for Git’s config semantics.
- **`test-tool genbytes`**: Junio suggested rebasing Kristofer Karlsson’s series to include a newly identified `test-tool genzeros | tr` callsite in `t/t1600-index.sh`.
- **`git archive` refactoring**: Karthik Nayak flagged a commit-message discrepancy in Srujith A Poojary’s series removing `USE_THE_REPOSITORY_VARIABLE` from the archive subsystem, calling it evidence of poor communication.