# Git mailing list daily digest for 2026/09/29

## Long threads

### stash: clean up index-mode test merge in `builtin/stash.c`

*Long thread (root 2026/09/19/21-26-34): 49 replies so far from 5 participants over 11 days; 8 messages that day.*

D. Ben Knoble posted v4 of the five-patch series "stash: clean up index-mode test merge", which fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true`. The final patch replaces the `diff-tree`/`apply --cached`/`reset` subprocess sequence in `git stash apply --index` with an in-core `merge_incore_nonrecursive()` call, fixing the autostash `MERGE_AUTOSTASH` lock bug. v4 drops the merge-verbosity changes, copies the resulting index merge tree OID only on success, adds a new patch 4/5 pinning `gc.reflogExpire` in t5520's setup, and moves the tree-order test into 5/5. Phillip Wood reviewed the range-diff and said everything except patch 4's commit message looks ready. Junio C Hamano pointed out a bug in v4 5/5: on a hard error `merge_incore_nonrecursive()` sets `result.clean` to -1, so `if (!result.clean)` is false and the else branch would dereference `result.tree`, suggesting a three-way check. Earlier, the thread had traced a t5520 failure to reflog expiry shifting fork-point detection, and Phillip Wood had resolved the confusion by showing `git pull --rebase` itself calls `git merge-base --fork-point`.

### `uploadpack.lazyFetchTrusted` config for server-side lazy fetching

*Long thread (root 2026/07/10/08-51-34): 31 replies so far from 3 participants over 82 days; 2 messages that day.*

Christian Couder's v4 series introduces the protected config variable `uploadpack.lazyFetchTrusted`, letting server operators mark repos whose git dir is trusted for lazy fetching when serving clones. Today Junio C Hamano suggested documentation wording fixes for the config, proposed an optional `:(optional)` marker for missing `safe.directory` paths (noting it is outside this series), and called the lazy-fetch control logic in `builtin/upload-pack.c` "trivially obvious and clear."

### push: fix `--force-if-includes` when remote-tracking ref has no reflog

*Long thread (root 2026/09/03/01-05-47): 23 replies so far from 6 participants over 27 days; 4 messages that day.*

Aleksei Sviridkin posted v4 of the fix initializing the timestamp to 0 in `is_reachable_in_reflog()`, so a remote-tracking ref without a reflog no longer cuts off the local reflog walk and wrongly rejects the push. v4 rewrites the log message to correct two errors and updates the t5533 test to use `setup_src_dup_dst` and expire only the remote-tracking reflog. Earlier today, Tyler Cipriani reported performance measurements and log-message problems, saying he would give a Reviewed-by once the message is clearer. Junio C Hamano said both v4 changes look sensible and will replace the patch and mark it for `next`.

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 20 replies so far from 4 participants over 11 days; 4 messages that day.*

Harald Nordgren posted v4 of the series using `remote.<name>.refmap` so a fresh remote in a shallow repository fetches nothing until a branch is tracked explicitly; v4 removes the automatic default-branch fetch, fixes the fetch report showing "new ref HEAD" instead of "new branch", and rewords the docs. Harald reiterated his objection that a bare `git fetch upstream` not fetching the HEAD branch does not make sense to him, hedging that "maybe that's for another topic". Junio C Hamano said "Will replace" for the v4 series and asked that `t/t5585-fetch-refmap.sh` be renumbered because that number is taken by another in-flight topic.

### branch: let `--delete-merged` find squash merged branches

*Long thread (root 2026/09/29/07-30-30): 10 replies so far from 3 participants over 1 day; 11 messages that day.*

Harald Nordgren posted a patch making `git branch --delete-merged` treat squash- or rebase-merged branches as merged when some upstream commit since the fork point contains all their changes, printing "Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)" with the first such commit; branches whose changes only partly landed are kept. D. Ben Knoble reviewed the approach, noting the check can be expensive, and floated a CSV-style option value; Harald said performance would be the only reason to hide the behavior and prefers the simplest command.

## The day in brief

D. Ben Knoble's v4 stash index-merge series and Jeff King's merge-ll tempfile cleanup both drew near-final review, with Junio C Hamano flagging one remaining bug in the stash patch where a hard error would dereference `result.tree` in the else branch. Aleksei Sviridkin's v4 `--force-if-includes` fix and Harald Nordgren's v4 shallow-fetch refmap series were both marked for replacement by Junio. Patrick Steinhardt posted a fix aligning reftable reflog timezone encoding with the specification after Josh McKinney reported the discrepancy.

## Notable threads

### refs/reftable: fix on-disk representation of reflog timezones
Patrick Steinhardt posted a 3-patch series fixing `refs/reftable-backend.c` to store reflog timezones as a signed offset in minutes per the reftable specification instead of the "[+-]HHMM" format, noting the fallout is limited because timezones are used only for display. The series adds `tz_to_minutes()`/`minutes_to_tz()` helpers in `date.c`, fixes a segfault in `test-tool dump-reftable -t`, and adds a test in `t/t0610-reftable-basics.sh`.

### filter-branch: fix commit map init from state branch
Grant Moyer's patch corrects `git-filter-branch.sh` populating the commit map dir from the state branch with an inverted from/to mapping, which produced errors with empty file names under `--prune-empty`. Michele Locati confirmed the bug, traced it to a change released in 2.50.0, tested the fix with `--state-branch`, `--prune-empty`, and `--subdirectory-filter`, and added a Tested-by tag.

### merge-ll: use tempfile API for external driver files
Jeff King posted v3 of "merge-ll: Cleanup merge driver temporaries after signal", switching `merge-ll.c` to the tempfile API so leftover `.merge_file_*` files are removed when Ctrl+C kills `git merge`, and catching `close()` errors when writing external tempfiles. Junio C Hamano called the patch very well reasoned and said he would replace it and mark it for `next`.

### use size_t for xdiff mmfile_t

*Long thread (root 2026/09/29/06-49-35): 8 replies so far from 3 participants over 1 day; 9 messages that day.*

Jeff King posted a 5-patch series consolidating `mmbuffer_t` into `mmfile_t`, switching buffer sizes to `size_t`, and simplifying `ll_ext_merge()` to use `read_mmfile()`. Junio C Hamano found that ignoring `read_mmfile()` errors could let `ll_ext_merge()` return `LL_MERGE_OK` with a NULL result that rerere's `merge()` does not check; Jeff King agreed, demonstrated a reproduction, and posted follow-ups [6/5] and [7/5] returning `LL_MERGE_ERROR` when reading the external merge result fails, with a rerere test.

### Reftable reflog timezone encoding differs from specification
Josh McKinney reported that Git stores reftable reflog timezone offsets as signed HHMM integers where the specification requires signed minutes, with a reproducer showing bytes `02 12` (530) instead of `01 4a` (330). Patrick Steinhardt agreed that sticking to the specification is the best way to resolve interoperability issues, called the fix somewhat urgent, and said he would aim to have it ready later this week.

## In brief
- Thomas Bachem posted v2 of a t5520 patch pinning `gc.reflogExpire`/`gc.reflogExpireUnreachable` to never; Junio C Hamano merged it into a synthetic base with the stash series in `seen`.
- Julia Evans' series removing `gittutorial-2` documentation; with it merged, doc-lint fails on `seen` with a "no link: gittutorial-2" error.
- Kazumasa Shigeta's patch exposing `-u`/`-a` in `git stash create`; Phillip Wood objected to the exit-status change and recommended `PARSE_OPT_STOP_AT_NON_OPTION`.
- Scott Chacon posted a 4-patch series porting sha1dc collision-detection acceleration to C, claiming roughly 2.7x faster hashing on a Xeon.
- RFC on optional per-repository consent before running local hooks; Junio C Hamano noted a tool that can install hooks could also approve them, and brian m. carlsson suggested `core.hooksPath` set to `/dev/null` as an equivalent.
- Pushkar Singh's v3 fixes swapped default reflog expiry periods in `reflog.h`; Patrick Steinhardt suggested an 88-day boundary for the regression test.
- Kaartic Sivaraam posted RFC v2 making setup.c append a "reason:" line to "not a git repository" errors when `--git-dir` is given explicitly.
- Patrick Steinhardt agreed to proceed with Jeff King's http patch handling curl stripping credentials from `CURLINFO_EFFECTIVE_URL`.
- Brigham Campbell posted v5 letting `git-contacts` read patches from stdin via `-`; Junio C Hamano asked whether the usage comment should also be updated.
- Stanislav Aleksandrov proposed a `^/` submodule URL syntax resolving from the server root of the superproject's remote.
- RFC on porcelain for sharing stashes through remotes; brian m. carlsson gave design feedback preferring explicit names and ref-only listing.
- **hostname: includeIf condition**: Jeff King said no one else is working on a `hostname:` includeIf condition and cited Ignacio Encinas's 2024 patches; Encinas confirmed dropping that series, and Hamano noted the thread was nearly finished.
