# Git mailing list daily digest for 2026/09/23

## Long threads

### `git repo info` path keys (GSoC series)

*Long thread (root 2026/07/16/01-21-31): 37 replies so far from 5 participants over 70 days; 3 messages that day.*

K Jayatheerth's series adds path keys to `git repo info` in `builtin/repo.c`: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, and `path.grafts` each with absolute and relative suffixes, plus `path.git-prefix` and `path.cdup`. Today he worked through review feedback on v6: he agreed the `if (!index_file)` checks in the `path.index` getters are dead code because `repo_get_index_file()` already BUG()s when the repository is not ready, and will remove them; on `path.cdup` he located the `is_inside_work_tree` branch in `rev-parse` that makes `git rev-parse --show-cdup` print an absolute path when the working directory is outside the work tree, and said adding that logic in `repo.c` should fix the discrepancy; and he agreed to split the `get_superproject_working_tree()` fix from the new `path.superproject-root` keys as Junio C Hamano suggested, while objecting that `t1500-rev-parse.sh` is not a good test location because the bug affects `repo` as well.

### sequencer leaves auto maintenance to the end of a sequence

*Long thread (root 2026/09/04/07-53-44): 32 replies so far from 5 participants over 20 days; 1 message that day.*

Thomas Bachem's series changes `sequencer.c` and the rebase/revert builtins so that auto maintenance runs once when a sequence completes rather than inside spawned commands: `disable_auto_maintenance()` appends `maintenance.auto=false` to `GIT_CONFIG_PARAMETERS` for spawned `git commit`, `git merge`, and exec commands, and `run_auto_maintenance()` is called from `builtin/rebase.c` and `builtin/revert.c` on success. v5 kept the code unchanged and only reworded commit messages and test comments. Today Phillip Wood reviewed the v5 test comments, said they make the tests easier to understand, and stated the series "looks ready for next to me".

### `http.sslVerifyStatus` for stapled OCSP responses

*Long thread (root 2026/08/11/17-02-00): 26 replies so far from 5 participants over 44 days; 4 messages that day.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so git validates stapled OCSP responses, defaulting to false because enabling it blocks connections to servers that send no stapled response. v7 combined the test approaches, keeping setup-free cases in `t5551` and responder-based cases in the new `t5585`. Today Patrick Steinhardt reviewed v7 and said he is happy with it apart from two commit-message paragraphs he finds excessive. SZEDER Gábor then reported that after the patch was merged to `next`, the last `t5585` test fails on his system (libcurl 7.81.0, Apache 2.4.52) with "server certificate verification failed", and questioned the `SSL_VERIFYSTATUS` prereq being satisfied by unrelated `git ls-remote` failures. Junio C Hamano reverted the topic out of `next` to give it time to mature before a reroll, and welcomed SZEDER's suggestion of a positive prereq check.

### regression fix for amending committed rebase conflicts

*Long thread (root 2026/09/23/13-16-43): 10 replies so far from 4 participants over 1 day; 11 messages that day.*

Patrick Steinhardt posted a regression fix for `builtin/rebase`: after 6257588252 made `git-commit` refuse `--amend` at a conflicted rebase stop, the refusal persisted even after the user committed the resolved conflict; his patch also requires `MERGE_MSG` in `sequencer_ongoing_operation()` and adds a test in `t/t3404-rebase-interactive.sh`. Phillip Wood reviewed it, agreed `MERGE_MSG` is probably the best available proxy, but asked whether cherry-pick, merge, and revert have the same problem. Elijah Newren warned that `MERGE_MSG` is unreliable because `git reset` removes it without moving HEAD, and proposed reverting the series for 2.56 or recording HEAD in `rebase-merge/stopped-head`. Junio C Hamano judged that restricting plain `git commit` would be post-3.0 work and preferred a clean revert over adopting the mechanism at -rc2; Patrick Steinhardt defended the direct-commit workflow and agreed reverting is the safest change for now.

## The day in brief
The `http.sslVerifyStatus` topic was reverted out of `next` after SZEDER Gábor reported a failing OCSP test on his system, and the rebase `--amend` regression sparked a debate in which Junio C Hamano and others leaned toward a clean revert of 6257588252 for Git 2.56. Phillip Wood called Thomas Bachem's sequencer auto-maintenance series ready for `next`.

## Notable threads

### `ls-files` reuse of the untracked cache
Tamir Duberstein's series lets `git ls-files` reuse the untracked cache that `git status` populates. He posted a v1 two-patch series with benchmarks showing a synthetic 100,000-file wildcard query dropping from 361 ms to 50 ms with a warm cache, retracted it as "operator error", then posted a revised v2 in three patches: fixing inconsistent ignore-file hashes, sharing the cache between `-unormal` and `-uall` via a new `dir_flags` value, and making `ls-files` write pending untracked cache and fsmonitor updates back to the index unless `--no-optional-locks` is given; updated benchmarks show the wildcard query at 33.3 ms.

### avoiding full fetches of new remotes in shallow repos
Harald Nordgren posted v2 of a patch making `git remote add` in an already-shallow repository use a special `+:` fetch refspec that fetches the branches local branches are built on plus the remote's default branch, instead of the wildcard refspec that makes plain `git fetch` unusably slow. Earlier in the day Phillip Wood, Junio C Hamano, and D. Ben Knoble discussed alternatives such as refusing remotes without `--track`, priming with `fetch = :`, or a `remote.<name>.refmap` configuration; Junio rejected the `+:` syntax as giving no clue that remote-tracking branches belong under `refs/remotes/<name>/` and said the patch should be split into bite-sized commits.

### submodule merge reads a commit from the wrong repository
Guillaume CHAUVEL reported that during a merge of a superproject with submodules, Git v2.56.0-rc2 sometimes tries to read from submodule A a commit that exists only in submodule B, producing "error: failed to merge submodule A (repository corrupt)" instead of a submodule conflict. His reproducer triggered the bug in all 50 Ubuntu 26.04 runs and 43 of 50 Alpine runs, and an AI analysis identified a likely cause: a stale delta-base cache entry surviving its pack's closure, where a pack from another submodule reusing the same `packed_git` address and base offset could return cached data from the wrong repository.

### stash index merge moved in-core
D. Ben Knoble posted v2 of a four-patch series fixing a bug reported by Eli Barzilay in the interaction between autostashing, staged index entries, and `stash.index=true`. Patch 4/4 replaces the file-system-based index merge (subprocesses `git-diff-tree`, `git-apply`, `git-reset`) with an in-core merge using `merge_incore_nonrecursive` and branch labels, which fixes the autostash bug where `git reset` attempted to lock a deleted `MERGE_AUTOSTASH` ref. v2 also gives the incore merge branch labels (without which merge-ort segfaults on conflicts), adds a test for conflicted index merges, and adds a `t7600` test for fast-forward merge with `--autostash`.

### reflog default expiry periods fix
Pushkar Singh reposted v2 of a fix for the swapped default reflog expiry periods in `REFLOG_EXPIRE_OPTIONS_INIT()`, so reachable entries expire after 90 days and unreachable entries after 30 days, with an expanded regression test in `t/t1410-reflog.sh` covering both cases. Patrick Steinhardt acknowledged the fix ("Oops, indeed. Thanks for finding this issue and fixing it!"), and Junio C Hamano reviewed v2, praising the commit message but questioning why the reachable cases use separate repository blocks while the unreachable cases share one.

## In brief
- Qin ShiCheng and Junio C Hamano discussed the layering of the kept-pack cache reset in the repack/pack-objects series; Qin proposed a `repo_invalidate_kept_pack_caches()` in `packfile.c` for v3, and both agreed the current downcast is not safe in the long term.
- Justin Tobler posted v2 of the object-file packfile flush fix, adapting the commit message and switching the test to a subshell per Patrick Steinhardt's review.
- Maciej Ciemborowicz posted v2 routing reference copy and rename through the transaction API so the reference-transaction hook observes the logical updates and concurrent changes are rejected.
- Christian Couder posted v2 of the early option scanning series, now using the command's own `struct option` array with a `PARSE_OPT_EARLY` flag, and converting `git fast-import`'s `--allow-unsafe-features` scan.
- Xavier Morel reported that git sends no basic auth on the second request when a credential helper returns an empty username with a non-empty password, because `init_curl_http_auth` exits early unless proactive auth is enabled.
- Maciej Ciemborowicz posted v5 of the refs old-OID reporting series for batched deletions, fixing the broken intermediate commits; Junio C Hamano confirmed all three patches build.
- Patrick Steinhardt endorsed the commit-reach patch that passes the repository explicitly into `can_all_from_reach()`, fixing the superproject/submodule commit-graph merge failure.
- Junio C Hamano noted that the git-p4 shell-metacharacter fix ignores the exit status of the diff|apply pipeline, unlike the original `system()` call.
- Jeff King objected to the "safe" naming for the proposed strbuf API, arguing it implies the normal API is unsafe and still would not be signal-handler safe.
- Harald Nordgren posted v3 of the shallow-clone `<rev>~N` hint patch, fixing a bug where chained `^` resolution past a shallow boundary printed the hint twice.
- Jeff King explained that the two-part AsciiDoc xref form controls the expanded link text across implementations, and warned that explicit anchors change user-visible HTML fragment URLs.
- Jeff King and Junio C Hamano debated working around Debian 12's curl 7.88.1 HTTP/2 authentication failures in CI, with Jeff proposing a version-based test prereq instead of a CI-job-name skip.
