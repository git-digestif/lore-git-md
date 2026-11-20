# Git mailing list daily digest for 2026/11/07

## Long threads

### fetch refmap and limited-fetch remotes in shallow repositories

*Long thread (root 2026/09/19/14-47-26): 66 replies so far from 5 participants over 50 days; 1 message that day.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository by adding `remote.<name>.fetchRefmap`, a `--limited-fetch` option to `git remote add`, and defaulting to limited fetch in shallow clones, with `@{push}` resolved through the refmap. Today D. Ben Knoble described his local notes workflow -- `core.notesRef` and `notes.rewriteRef` set to `refs/notes/benknoble/commits`, `notes.displayRef` set to `refs/notes/origin/amlog`, and a negative refspec excluding amlog notes from pushes -- and argued that a `pushRefmap` would let him separate default push selection (`push.default`) from ref mapping for specific pushes such as notes, while cautioning that force-push commands must not forcibly overwrite remote notes.

### path keys for `git repo info`

*Long thread (root 2026/07/16/01-21-31): 53 replies so far from 6 participants over 115 days; 2 messages that day.*

K Jayatheerth's GSoC series exposes repository paths and metadata through `git repo info`, adding keys such as `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, and `path.cdup`, and refactoring `get_superproject_working_tree()` to take a `struct repository *`. Today Jayatheerth posted v8 of the 7-patch series, dropping `path.grafts`, moving the `--git-dir` test to `t/t1500-rev-parse.sh`, and special-casing `/dev/null` in `path.hooks` so it is printed directly. Phillip Wood reviewed the series, suggesting `path.working-tree.*` instead of `path.toplevel.*`, `path.superproject-working-tree.*` instead of `path.superproject-root.*`, and `path.prefix` instead of `path.git-prefix`, questioning whether `path.hooks` is needed given config-based hooks, and asking how `path.cdup` differs from `path.working-tree.relative`.

### `%gb` placeholder naming the branch of each reflog entry

*Long thread (root 2026/10/15/13-13-55): 34 replies so far from 6 participants over 24 days; 2 messages that day.*

Harald Nordgren's series adds a `%gb` placeholder to `git log -g --format` that names the branch `HEAD` was on when each reflog entry was recorded, showing "detached at <commit>" when detached, implemented in `reflog-walk.c` and `pretty.c`. Today Nordgren posted v5: "detached at <commit>" now names the commit `HEAD` was detached at (or the commit a detached rebase ended on), `git rebase <upstream> <rev>` counts `<rev>` as a branch only if `refs/heads/<rev>` exists, a commit whose subject starts with "checkout" is no longer mistaken for a rebase checkout, and tests were added for a detached rebase that fast-forwards and one that is aborted.

### detached HEAD documentation rewrite

*Long thread (root 2026/10/07/14-52-01): 21 replies so far from 4 participants over 32 days; 1 message that day.*

Julia Evans's patch rewrites the detached HEAD explanation into a standalone `git help detached-head` guide wired into `advice.c` and `command-list.txt`, addressing why detached HEAD is useful, why users end up there accidentally, and `git switch`'s harder-to-detach behavior. Today D. Ben Knoble reiterated interest in landing the manual, naming two remaining blockers -- a reroll with the SQUASH??? commit applied and resolution of how to describe `HEAD` -- proposed merging v4's second paragraph with Junio C Hamano's points about not clobbering branches, and concluded the thread is approaching a finish.

## The day in brief

Thomas Bachem submitted a 15-patch series fixing segfaults from valueless config settings across rebase, fetch, grep, blame, merge, and config listing. Tanga Joris diagnosed a split-index plus skipHash corruption where the shared index is named after a zeroed checksum. Harald Nordgren posted v5 of the reflog `%gb` series, and K Jayatheerth posted v8 of the `git repo info` path-keys series.

## Notable threads

### segfaults on settings given without a value

Thomas Bachem submitted a 15-patch series fixing crashes when a setting is given without a value (e.g. `git -c foo.bar`), covering `rebase.missingCommitsCheck`, `core.alternateRefs*`, interactive patch color slots, `fetch.bundleCreationToken`, `grep.patternType`, `remote.*.tagOpt`, `color.blame.*`, `branch.*.mergeOptions`, `remotes.*`, `safe.bareRepository`, `bitmapPseudoMerge.*.pattern`, `scalar.repo`, `remote.*.url` in hasconfig includes, bundle list keys, and `git config list --type=color`/`--type=expiry-date` (a regression from 2.54.0).

### splitIndex with skipHash corrupts the index

With `core.splitIndex` and `index.skipHash` both enabled, index entries are lost and `git status` crashes. Today Tanga Joris reproduced both cases on master and diagnosed the cause: `write_shared_index()` names the file after the zeroed trailing checksum (`sharedindex.0000...`), and the null base makes `read_index_from()` treat the shared index as absent. He said he is working on a fix; Andy Koppe confirmed he had not started one.

### `git merge --no-overwrite-ignore` for real merges

Aniket Gupta submitted a patch adding a `preserve_ignored` bit to `struct merge_options` so `git merge --no-overwrite-ignore` aborts instead of overwriting ignored files during real merges -- previously honored only for fast-forwards -- wiring the option through the trivial-merge path in `builtin/merge.c` and merge-ort's checkout, with tests for a plain merge and a directory rename.

### `git apply --check` crash on malformed diff headers

Tanga Joris posted an Outreachy two-patch series fixing the `check_preimage` assertion crash in `git apply --check`: the first frees names from an unused `diff --git` line when a non-git-header line follows, and the second rejects new and deleted files that carry both names.

### sideband allowControlCharacters keyword loop

Thomas Bachem's patch makes the `sideband.allowControlCharacters` parser advance past an unrecognized keyword after warning, instead of warning forever on every fetch, push, clone, and ls-remote. Junio C Hamano reviewed it, judged the change reasonable, endorsed the testing method, and marked the topic ready for `next`.

## In brief

- Junio C Hamano noted `checkout_fast_forward()` still calls `unpack_trees()`, which depends on `the_repository`, and attached an RFC making `unpack-trees.c` compile without `USE_THE_REPOSITORY`; he marked Okhuomon Ajayi's merge.c cleanup ready for `next`.
- Junio C Hamano reported the Coverity overflow series has been waiting for response since mid-October and will expire from his tree without further activity.
- Ravi Mistry posted v3 of the HTTP retry series defaulting `http.retryAfter` to 10 seconds; Junio C Hamano questioned implementing 0-second waits first and changing the default in the final patch.
- Junio C Hamano posted a two-patch series removing `USE_THE_REPOSITORY` from `unpack-trees.c` via an `unpack_repo()` helper, plus a consistency rename of the `index` parameter to `istate`.
- D. Ben Knoble proposed making `git diff-tree` handle format-rev's "revs" mode and restricting format-rev to "text" mode.
- Aniket Gupta confirmed Hannes's report -- a pure rename followed by an edit fails under `core.fileMode=false` in a single `git apply` -- is valid format-patch input that worked before 0482c32c33.
- Junio C Hamano retracted his readiness mark on the describe `--debug` patch after noting `prio_names[]` is still wrapped in `_()`.
- Kristofer Karlsson posted a series adding `test-tool genbytes` to avoid slow `tr(1)` on macOS; Junio C Hamano suggested squashing the rename patch and using hex byte values.
- Junio C Hamano marked Okhuomon Ajayi's merge-blobs cleanup ready for `next`.
- Jelle van der Waa posted an RFC adding `--verbose` to `git history reword`; D. Ben Knoble suggested a different name such as `--commit-verbose`.
- Tanga Joris suggested shrimech pick up a commit making `ObjectID` members private for the invalid-algorithm formatting fix.
- Hussain Sitamouwala documented promisor remote fetch ordering in `partial-clone.adoc`; Junio C Hamano asked whether it documents long-standing behavior.
