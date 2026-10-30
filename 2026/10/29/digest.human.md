# Git mailing list daily digest for 2026/10/29

## The day in brief
A regression in `git grep --recurse-submodules` was reported and bisected, the `fetch: introduce +: refspec` series inched closer to resolution with a maintainer-approved design pivot, and Junio C Hamano queued over a dozen patches for integration after Patrick Steinhardt’s blanket approvals. The notes subsystem’s concurrency model sparked a lively debate about retry mechanisms, while a new bugfix for shallow fetches with tags was merged to `seen`.

---

## Notable threads

### Regression in `git grep --recurse-submodules` when searching tree objects
**What changed?**
A regression in `git grep --recurse-submodules` causes the command to fail with `fatal: unable to read tree` when it encounters a directory inside a submodule. The issue was introduced in Git 2.56.0 by commit `93cd344e34` ("builtin/grep: stop registering submodule ODB as source"), which removed submodule ODB registration but did not update `grep_tree()` to read from the correct repository context.

### Why it matters

The regression breaks recursive searches of submodule trees, a workflow used in CI environments and large monorepos. The working-tree form of the command remains unaffected, but the tree-object form is now unusable for submodules containing directories.

### Today’s exchange

Root reported the regression, provided a minimal reproduction script, and bisected it to the problematic commit. The proposed workaround—reading from `repo->objects` instead of `the_repository->objects`—restores expected behavior but has not been tested against the full test suite. No maintainer response yet, but the bisect and reproduction steps make this a high-priority fix.

---

### `fetch: introduce +: refspec` series: override semantics confirmed
**What changed?**
The series, which introduces `remote.<name>.fetchRefmap` to decouple fetch selection from namespace mapping, is now aligned on **override semantics**: command-line refspecs (or colon-separated `remote.<name>.fetch` entries) must override `fetchRefmap` entirely, not coexist with it. This resolves the ambiguity raised by Phillip Wood and aligns with Junio C Hamano’s latest feedback.

### Why it matters

The design pivot ensures backward compatibility and avoids silent behavior changes for users migrating from older configurations. The resolution clears the last technical blocker for the series, which is now poised for integration.

### Today’s exchange

- **Junio C Hamano** acknowledged the ambiguity in the current design but left the door open for Phillip’s override semantics, stating it “could be ‘refrain from what the refmap alone would do and do something else’.”
- **Phillip Wood** insisted that command-line refspecs must override `fetchRefmap`, providing the concrete example:
  ```sh
  git -c remote.cabal.fetchRefmap=... fetch cabal refs/heads/master:refs/remotes/origin/master
  ```
  This should update only `origin/master`, not `cabal/master`. The thread now awaits a revised patch from Harald Nordgren to implement the agreed-upon behavior.

---

### Notes subsystem: retry mechanism debated
**What changed?**
A patch fixing a race condition in `git notes add` (where concurrent updates could silently overwrite each other) sparked a debate about the notes subsystem’s concurrency model. The current patch fails on *any* concurrent update, but Junio C Hamano and Kristoffer Haugsbakk proposed a **retry mechanism** using Git’s existing merge/rebase machinery to allow non-conflicting updates to proceed.

### Why it matters

The notes subsystem is used in workflows like Gerrit and GitHub for code review metadata, where concurrent updates are common. The current patch’s strict failure-on-concurrency behavior is overly restrictive for non-conflicting updates (e.g., two processes adding notes to unrelated commits).

### Today’s exchange

- **Kristoffer Haugsbakk** argued that the current patch is overly strict and proposed a retry mechanism that only fails if updates target the same commit.
- **Junio C Hamano** sketched a concrete alternative: losing processes could merge/rebase their attempted note commits on top of the winning process’s commit, leveraging Git’s existing merge machinery.
- **Farid Zakaria** deferred implementing the retry mechanism, positioning the current patch as a “net-positive” interim fix. He also teased two additional race-condition bugs in `credential-cache` and `fsmonitor`, hinting at broader concurrency issues in Git.

---

### Shallow fetch with tags: bugfix merged to `seen`
**What changed?**
A bugfix for shallow fetches with tags, which caused clients to report no ‘have’ references during negotiation (forcing servers to send all objects), was merged to Junio’s `seen` branch. The fix exposes pending transaction state via the transport’s `negotiation_include_tips` field, restoring correct ‘have’ reporting while preserving atomicity for `--atomic` fetches.

### Why it matters

The regression, introduced by commit `0e358de64a` ("fetch: use batched reference updates"), degraded performance for shallow fetches with tags. The fix is a high-priority correction for users of the compatibility-hash feature.

### Today’s exchange

- **Junio C Hamano** merged the v3 series to `seen`, noting a minor integration hiccup: the original `repo_write_loose_object_map()` function loses all callers and will disappear, while the new helper `loose_object_map_write()` survives.
- **Justin Tobler** posted v3, incorporating a locking-scope optimization (reading the source loose-object-idx file outside the lock) and a clarifying comment for `loose_object_idx_merge()`.

---

### `git worktree list` filtering: design pivot to `for-each-ref`
**What changed?**
The discussion about adding filtering to `git worktree list` pivoted from a `--branch` option to extending `git for-each-ref`’s `%(worktreepath)` atom to report worktree paths even during rebase/bisect. This narrower change avoids introducing new CLI options but introduces new concerns about safety and composability.

### Why it matters

The pivot simplifies the design but raises questions about whether `for-each-ref` is the right tool for filtering worktrees. Key limitations include:
- No NUL-terminated output, complicating parsing of worktree paths with newlines.
- Inability to distinguish the main worktree from linked worktrees, risking main repository deletion.
- Risk of surfacing worktrees in transient, high-risk states (e.g., mid-rebase), which could lead to data loss if used with `git worktree remove`.

### Today’s exchange

- **Jasper Mayone** proposed the pivot, framing it as a bug fix to align `%(worktreepath)` with existing Git behavior.
- **Eric Sunshine** highlighted a latent design tension: filtering worktrees breaks the invariant that the main worktree appears first in output, creating a risk for scripts that rely on this ordering.
- **Phillip Wood** identified three key limitations of `for-each-ref` as a filtering mechanism and endorsed the behavior change as a bug fix.
- **Ben Knoble** raised a critical edge case: whether filtering worktrees by branch state (e.g., `--merged=origin`) should include worktrees that are mid-rebase, and whether `git worktree remove` should allow deletion of such worktrees.

---

## In brief
- **`midx: avoid publishing empty incremental layers`**: Junio C Hamano queued Pia Park’s v3 patch, which prevents MIDX writes from publishing empty layers (zero objects) without a reverse index. The patch is unchanged from v2 except for test refinements.
- **`doc: sync command-list.txt`**: Tuomas Ahola’s v3 series, which keeps `command-list.txt` synchronized with non-command manual pages, exempts the obsoleted `gittutorial-2(7)` manual page from the linter check.
- **`branch --delete-merged: detect squash-merged and rebase-merged branches`**: Jasper Mayone’s tested review confirmed the v4 series behaves as documented: `--delete-merged` preserves strict tip-reachability, while `--delete-integrated` correctly deletes squash merges, GitHub-style squashes, rebase merges, and upstream-modification cases.
- **`completion: add --graph-indent`**: Pablo Sabater questioned whether `--graph-indent` and `--no-graph-indent` belong in `__git_log_common_options` and proposed adding `--graph-lane-limit`. He also flagged a UX concern: both options require `--graph` to be present, and autocompleting them without `--graph` will produce an error.
- **`sigchain: repair signal handler chain`**: Phillip Wood offered two optimizations for the thread-level fix: block signals once before thread-creation loops (e.g., `preload_index()`) and unblock after, and mask at least SIGINT/SIGHUP/SIGTERM/SIGQUIT during handler execution.
- **`test modernization in t5300-pack-object.sh`**: Junio C Hamano queued Ogundipe Sola Taiwo’s patch, which replaces five legacy file-check idioms with modern helpers (`test_path_is_file`, `test_path_is_missing`).
- **`Add degree-of-concern reporting to git repo structure`**: Junio C Hamano identified a copy-paste error in the v4 series: the concern level for the largest tree size is incorrectly computed using `objects->largest.commit_size.value` instead of `objects->largest.tree_size.value`.
- **`completion: remove NO_PARSEOPT`**: Tanga Joris posted v3 of their series, implementing Junio’s proposed refactoring to centralize `--git-completion-helper` handling in `parse-options.c`.
- **`oss-fuzz: fix loose-object-idx migration`**: Patrick Steinhardt endorsed Sami Jawhar’s one-line fix, which sets a placeholder `.git` gitdir in the fuzzer’s repository to avoid a BUG in `prepare_repo_settings()`.
- **`reflog: fix dry-run output inconsistency`**: Patrick Steinhardt’s surface-level review approved Laveesh Bansal’s patch, noting the bug was only a messaging issue (no functional regression).
- **`What's cooking in git.git` (October 2026 #10)**: Junio’s report advanced 13 topics to `next`-ready status after Patrick Steinhardt’s blanket approvals. Key graduations:
  - `hn/imap-send-header-fix`: Approved for `next` after Patrick’s substantive review.
  - `kn/fetch-tag-backfill-batch-fix`: Queued for `next` after Karthik Nayak’s final reroll.
  - `mc/refs-rename-transaction`: Evicted due to AI-heavy implementation and unclear author understanding.
- **`notes: fix race condition in concurrent note additions`**: Junio C Hamano requested rephrasing of Farid Zakaria’s ambiguous statement about AI-assisted review. Farid clarified that he was expressing confidence in his own understanding of the patch.
- **`redesigning signal handling in threaded Git code`**: ian4hu revised the design to block `SIGINT`, `SIGHUP`, `SIGTERM`, `SIGQUIT`, and `SIGPIPE` during thread creation using a guard pattern, preserving `SIGPIPE` for the main thread.
- **`t0410: use test_path_exists`**: Junio C Hamano queued Tilak Kumar’s patch, which replaces a raw `test -e` check with `test_path_exists` in `t/t0410-partial-clone.sh`.
- **`git check-ref-format` refactoring**: Junio C Hamano identified four issues in Husain Sitamuwala’s patch: confusing usage-string names, `--print` not marked hidden, silent acceptance of negated options, and retention of `show_usage_if_asked()` that defeats `parse-options` help generation.