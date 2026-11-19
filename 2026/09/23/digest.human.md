# Git mailing list daily digest for 2026/09/23

## Long threads

### `git repo info` path keys series

*Long thread: 37 replies so far from 5 participants over 70 days; 3 messages that day.*

K Jayatheerth's GSoC series adds seven path keys to `git repo info` in `builtin/repo.c` (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, `path.cdup`), each with absolute and relative suffixes. Today he agreed to drop the dead `if (!index_file)` checks in the `path.index` handlers, since `repo_get_index_file()` already BUG()s on an unready repository. On `path.cdup` he found the `is_inside_work_tree()` branch in `builtin/rev-parse.c` behind `git rev-parse --show-cdup` printing an absolute path outside the work tree, and said porting it to `builtin/repo.c` should fix the discrepancy. He also agreed to split the `get_superproject_working_tree()` fix from the new `path.superproject-root` keys, but objected to testing the fix in `t1500-rev-parse.sh` since the bug affects `git repo` too.

### Sequencer auto-maintenance timing

*Long thread: 32 replies so far from 5 participants over 20 days; 1 message that day.*

Thomas Bachem's series makes the sequencer leave auto maintenance to the end of a sequence: `disable_auto_maintenance()` in `sequencer.c` appends `maintenance.auto=false` to `GIT_CONFIG_PARAMETERS` for spawned commit, merge and exec commands, while `builtin/rebase.c` and `builtin/revert.c` run auto maintenance once when the sequence completes. Phillip Wood reviewed the v5 test comments and said the series "looks ready for next to me".

### `http.sslVerifyStatus` OCSP stapling check

*Long thread: 26 replies so far from 5 participants over 44 days; 4 messages that day.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` to validate stapled OCSP responses, defaulting to false because enabling it blocks servers that send no stapled response. Patrick Steinhardt reviewed v7, happy apart from two excessive commit-message paragraphs. After the topic was cooking in `next`, SZEDER Gábor reported the last `t5585` test failing on his system and questioned the `SSL_VERIFYSTATUS` prereq; Junio C Hamano reverted the topic out of `next` to let it mature.

### Rebase `--amend` regression after conflict commit

*Long thread: 10 replies so far from 4 participants over 1 day; 11 messages that day.*

Patrick Steinhardt posted a regression fix for `builtin/rebase`: since 6257588252 made `git commit` refuse `--amend` at a conflicted rebase stop, the refusal persisted after the user committed the resolution; his patch also requires `MERGE_MSG` in `sequencer_ongoing_operation()`. Phillip Wood said the code looks good but asked whether cherry-pick, merge and revert share the problem; Elijah Newren warned `MERGE_MSG` is unreliable because `git reset` removes it without moving HEAD. The discussion converged on reverting 6257588252 for Git 2.56, which Junio C Hamano preferred at -rc2.

## The day in brief
Junio C Hamano reverted the `http.sslVerifyStatus` topic out of `next` after SZEDER Gábor reported a failing OCSP test, while Phillip Wood called the sequencer auto-maintenance series ready for `next`. The rebase `--amend` regression discussion leaned toward reverting the regressing commit for 2.56.

## Notable threads

### Repack kept-pack snapshot
Qin ShiCheng's series stops `git repack -d` from deleting a pack whose objects were never copied when a `.keep` appears mid-run, by handing pack-objects repack's startup snapshot. He argued the kept-pack cache invalidation belongs in `packfile.c` as `repo_invalidate_kept_pack_caches()`; Junio C Hamano accepted the point while noting the layering violation remains until packfile management moves to the files backend.

### Submodule merge reads wrong repository
Guillaume CHAUVEL reported that merging a superproject with submodules sometimes makes Git v2.56.0-rc2 read from submodule A a commit existing only in submodule B, failing with "repository corrupt" instead of a submodule conflict; his reproducer triggered it in all 50 Ubuntu and 43 of 50 Alpine runs, with an AI analysis pointing to a stale delta-base cache entry outliving its pack.

### Early option scanning API
Christian Couder posted v2 adding an `early_scan_options()` sub-API to parse-options that reuses the command's `struct option` array via a `PARSE_OPT_EARLY` flag, and converted `git fast-import`'s hand-rolled scan so `--allow-unsafe-features` after `--depth 5` is no longer silently ignored. Junio C Hamano agreed that converting `bisect_start()` to parse-options would make `--` detection come for free.

## In brief
- Harald Nordgren posted v2 making `git remote add` use a `+:` refspec in shallow repos; Junio C Hamano rejected the syntax, proposing a `remote.<name>.refmap` config instead.
- Tamir Duberstein posted v2 letting `ls-files` reuse and update the untracked cache; a wildcard query benchmark dropped to 33.3 ms warm.
- Maciej Ciemborowicz posted v2 routing ref copy and rename through the transaction API so the reference-transaction hook sees the logical updates.
- Justin Tobler posted v2 fixing the packfile flush during ODB transaction commit with batch loose-object fsync.
- Patrick Steinhardt endorsed passing the repository explicitly into `can_all_from_reach()`, fixing the superproject/submodule commit-graph merge failure.
- Junio C Hamano noted the git-p4 shell-metachar fix drops the exit-status check of the diff|apply pipeline.
- Pushkar Singh reposted v2 fixing the swapped default reflog expiry periods; Patrick Steinhardt acknowledged it.
