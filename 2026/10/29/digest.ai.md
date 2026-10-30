# Git mailing list daily digest for 2026/10/29

## The day in brief
The Git project saw significant progress on several fronts today. A critical bugfix for the multi-pack-index subsystem was queued for integration, resolving a regression that caused bitmap-enabled writes to fail. The `git fetch` refspec redesign reached a pivotal moment with maintainer feedback on override semantics, while a long-running refactoring of `git rerere` to use standard subcommand handling was finalized. Performance regressions in shallow fetches and quarantine alternates received targeted fixes, and the ongoing `the_repository` removal effort continued with several patches advancing toward integration.

## Notable threads

### midx: avoid publishing empty incremental layers
**What changed**: Junio C Hamano queued Pia Park's v3 patch that prevents MIDX writes from publishing empty layers (zero objects) without a reverse index. This bugfix addresses a regression where subsequent `--bitmap` writes would fail when attempting to load the missing reverse index for empty layers.

**Why it matters**: The multi-pack-index (MIDX) subsystem is performance-critical for large repositories. This fix ensures correct behavior when using incremental MIDX updates with bitmaps, preventing failures in common maintenance operations.

**Technical details**: The patch modifies `midx-write.c` to exit early in `write_midx_internal()` when no objects are found, preserving silent success (exit status 0) for compatibility. Three new regression tests in `t/t5334-incremental-multi-pack-index.sh` cover edge cases including empty initial layers, empty packs, and packs with only duplicate objects.

**Current status**: Queued for integration with substantive approval from Taylor Blau and Junio's procedural confirmation ("Will queue").

---

### fetch: introduce `+:` refspec for matching-inspired fetch mode
**What changed**: The design discussion for Harald Nordgren's fetch refspec redesign reached a critical juncture. Junio C Hamano and Phillip Wood engaged in substantive discussion about how command-line refspecs should interact with the new `remote.<name>.fetchRefmap` configuration.

**Why it matters**: This series aims to resolve a long-standing performance issue in shallow repositories where `git fetch` would negotiate history for all remote branches. The current design debate will determine whether users face silent behavior changes (dual storage) or explicit overrides when migrating from older configurations.

**Technical details**: Phillip Wood insists that command-line refspecs must override `fetchRefmap` entirely, not coexist with it. His example demonstrates that `git -c remote.cabal.fetchRefmap=... fetch cabal refs/heads/master:refs/remotes/origin/master` should update only `origin/master`, not `cabal/master`. Junio acknowledges this could be the correct interpretation but notes the current behavior treats colon-separated refspecs as additive.

**Current status**: The discussion remains unresolved but leans toward Phillip's override semantics. The resolution will require updates to `remote.c` (precedence logic in `get_ref_map()`) and documentation.

---

### branch --delete-merged: detect squash-merged and rebase-merged branches
**What changed**: Jasper Mayone provided a tested review of Harald Nordgren's v4 series that adds `--delete-integrated` to `git branch`. The review confirms the series behaves as documented across various real-world scenarios.

**Why it matters**: This feature provides practical repository maintenance for common forge workflows (GitHub-style squash merges, rebases) that don't leave traditional merge commits. The tested review provides confidence in the implementation's correctness.

**Technical details**: Jasper's testing confirms that `--delete-merged` preserves strict tip-reachability (including Junio's false-positive case), while `--delete-integrated` correctly deletes squash merges, GitHub-style squashes, rebase merges, and cases where upstream modified the same file before squashing. The output format consistently names the correct landing commit.

**Current status**: The series is functionally sound but still requires implementation of the author-check mitigation to address false positives in forge workflows.

---

### tmp-objdir: fix loose-object-idx migration with compat hash + batch fsync
**What changed**: Justin Tobler's bugfix series for ODB transaction layer issues was merged to `seen`. The series fixes a critical migration failure in the "loose-object-idx" file when Git is configured with compatibility hashes and batch fsync.

**Why it matters**: This bug caused ODB transactions to fail with "files differ in contents" errors, risking data loss for users of the compatibility-hash feature. The fix is high-priority for ODB stability.

**Technical details**: The v3 series implements atomic writes of the merged index and uses a finite 1000ms lock timeout. The fix touches `tmp-objdir.c`, `loose.c`, and `loose.h`, with a new regression test in `t/t1016-compatObjectFormat.sh`. The series addresses all blocking concerns from Patrick Steinhardt, including atomicity and locking.

**Current status**: Merged to `seen` with Junio noting a minor integration hiccup (the original `repo_write_loose_object_map()` function loses all callers).

---

### What's cooking in git.git (October 2026 #10)
**What changed**: Junio C Hamano's integration report showed significant progress, with 17 topics graduating to `master` and 13 advancing to `next`-ready status. Notable graduations include sequencer fixes, config enhancements, CI housekeeping, and rerere robustness improvements.

**Why it matters**: The report provides visibility into the project's integration pipeline and highlights stalled topics at risk of being dropped. Several long-running efforts (the_repository removal, rerere libification, CI security hardening) saw substantial progress.

**Technical details**: Key graduations to `master`:
- `ak/sequencer-head-refdup`: Sequencer fixes for worktree lookup
- `ic/includeif-hostname`: New `includeIf.hostname:` config condition
- `ps/ci-housekeeping-and-modernizations`: CI updates including Python 2 removal
- `tb/rerere-wait-for-merge-rr-lock`: Rerere robustness improvements

Notable new topics ready for `next`:
- `tb/ci-pin-actions`: CI security hardening
- `ps/rerere-subcommands`: Rerere subcommand refactoring
- `kn/fetch-tag-backfill-batch-fix`: Tag backfill negotiation fix
- `hn/imap-send-header-fix`: IMAP-send header handling improvements

**Current status**: The integration pipeline shows healthy progress, with several long-running efforts nearing completion.

## In brief
- **doc: sync command-list.txt with non-command manual pages**: Tuomas Ahola's v3 series exempts the obsoleted `gittutorial-2(7)` manual page from linter checks and incorporates all prior feedback.
- **completion: add --graph-indent and --no-graph-indent to git log**: Pablo Sabater questions whether `--graph`-dependent options should be grouped separately and proposes adding `--graph-lane-limit`.
- **sigchain: repair signal handler chain when empty but previously pushed**: Phillip Wood offers optimizations for the thread-level fix, including blocking signals once before thread-creation loops.
- **test modernization in t5300-pack-object.sh**: Ogundipe Sola Taiwo's patch was queued for integration after addressing commit message feedback.
- **Add degree-of-concern reporting to `git repo structure`**: Junio identified a copy-paste error in Mark C. Chu-Carroll's v4 series that incorrectly computed tree size concern levels.
- **`git worktree list --merged` / `--no-merged`**: The design discussion continues with substantive feedback about the limitations of using `for-each-ref` as a filtering mechanism.
- **remote: align @{push} resolution with push.default=simple in triangular workflows**: Junio raised structural concerns about the implementation and suggested refactoring for readability.
- **bundle-uri: use passed repository parameter**: Okhuomon Ajayi's v2 patch eliminates all remaining uses of `the_repository` in `bundle-uri.c`.
- **odb.c: skip self-referential alternates during quarantine**: Justin Tobler proposed an ODB transaction callback as a cleaner architectural fix for the quarantine alternate-ref performance bug.
- **range-diff: use repository from diffopt instead of the_repository**: Okhuomon Ajayi's patch was queued for integration after addressing commit message feedback.
- **notes: fix race condition in concurrent note additions**: Farid Zakaria deferred implementing a retry mechanism, positioning the current patch as an interim fix.
- **redesigning signal handling in threaded Git code**: ian4hu revised the design to block signals during thread creation using a guard pattern.
- **t0410: use test_path_exists instead of test -e**: Tilak Kumar's test modernization patch was queued for integration.
- **git check-ref-format option precedence**: Junio identified several issues with Husain Sitamuwala's refactoring to use `parse-options`.
- **regression in `git grep --recurse-submodules` when searching tree objects**: Root reported a regression in `git grep --recurse-submodules` when searching tree objects, bisected to commit `93cd344e34`.