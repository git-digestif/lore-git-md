# Git mailing list daily digest for 2026/10/01

## Long threads

### stash: in-core index merge for `stash apply --index`

*Long thread (root 2026/09/19/21-26-34): 53 replies so far from 5 participants over 13 days; 2 messages that day.*

D. Ben Knoble's series fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true`, where `git stash apply --index` used a subprocess dance of `git diff-tree`, `git apply --cached`, and `git reset`. The latest revision replaces that with an in-core `merge_incore_nonrecursive()` call in `builtin/stash.c`, with a tri-state check distinguishing `result.clean < 0` (hard error) from `!result.clean` (conflict). Earlier rounds drew review on a NULL-dereference risk and a t5520 reflog-expiry interaction. Today Phillip Wood confirmed the v5 range-diff looks as expected and expressed pleasure at removing subprocesses from `git stash`, and Junio C Hamano said the series looks very good to him and marked it for `next`.

### refs: report old values to reference-transaction hooks

*Long thread (root 2026/09/19/13-34-15): 29 replies so far from 5 participants over 13 days; 1 message that day.*

Maciej Ciemborowicz's v6 fixes reference-transaction hooks reporting an all-zero old OID for unconditional updates with a single change in the common transaction layer in `refs.c`, resolving old values for the hook before "preparing" and refreshing them after the backend locks the refs, only when a hook exists; earlier versions had taken a caller-specific three-patch approach that reviewers argued was the wrong layer. Today Maciej lowered the priority of the bug after resolving his branch and tag deletion issues using data returned by the transaction in the prepared state, identified branch renaming as the remaining issue he cannot reasonably address without this fix, and asked for more attention to that bug.

### rerere: wait for MERGE_RR.lock and skip a held lock in auto gc

*Long thread (root 2026/09/02/08-31-37): 23 replies so far from 4 participants over 30 days; 2 messages that day.*

Thomas Bachem's series makes every rerere writer wait `rerere.lockTimeout` for `MERGE_RR.lock`, adds a skip for auto-maintenance gc, and lets conflict-time callers warn and go on. Patrick Steinhardt had proposed renaming the option to `--skip-locked` as a hidden, undocumented flag. Today Thomas agreed to remove caller documentation from git-rerere(1), rename the option to `--skip-locked`, rename patch 3's `RERERE_SKIP_LOCKED` flag to `RERERE_WARN_LOCKED`, and reported that calling `rerere_gc()` directly from `builtin/gc.c` would die with exit code 128 on a held lock instead of reporting a failed maintenance task, so he plans to keep the option; Patrick agreed to keep the hidden option.

### Documentation: new gitmergeconflicts man page

*Long thread (root 2026/09/24/14-44-15): 23 replies so far from 6 participants over 8 days; 2 messages that day.*

Julia Evans's series adds `Documentation/gitmergeconflicts.adoc`, a guide to resolving merge conflicts, and links it from `git-merge`, `git-cherry-pick`, `git-rebase`, `git-revert`, and `git-pull`. Today Patrick Steinhardt proposed extending conflict-marker labels to include the commit ID, commit message, and date, arguing it would spare users from learning refs like `MERGE_HEAD`; he said he would open a GitLab issue to have someone try it, and agreed `merge.conflictstyle=diff3` should become the default. Julia Evans endorsed including dates and commit messages as very helpful.

### xdiff: use size_t for mmfile_t and consolidate mmbuffer_t

*Long thread (root 2026/09/29/06-49-35): 21 replies so far from 4 participants over 3 days; 4 messages that day.*

Jeff King's v2 series cleans up `read_mmfile()` error handling, replaces `mmbuffer_t` with `mmfile_t`, switches buffer sizes to `size_t`, NUL-terminates buffers, and makes `merge-ll` use `read_mmfile()` for external merge results, reporting an error when the read fails. Today Patrick Steinhardt flagged a commit-message nit on the NUL-termination patch and said "Okay" on the `merge-ll` patch, and Junio C Hamano accepted keeping the name `mmfile_t` and agreed the error-reporting fix is much easier after the `read_mmfile()` switch because there is a single spot to error-check.

## The day in brief

Junio C Hamano marked D. Ben Knoble's in-core index merge for `git stash apply --index` for `next` and said he will mark Patrick Steinhardt's reftable reflog-timezone fix for `next`. Separately, Coy Geek reported that merge-ort's `--no-overwrite-ignore` protection is bypassed when directory rename detection targets an ignored file, and Patrick Steinhardt confirmed the delta-base cache as the cause of a submodule merge bug.

## Notable threads

### merge-ort: `--no-overwrite-ignore` bypassed at directory-rename destination
Coy Geek reported that `git merge --no-overwrite-ignore` overwrites an ignored, untracked file when directory rename detection moves a committed file to that path: with `merge.directoryRenames=true` the merge succeeds and overwrites the content, and with the default `conflict` mode it stops with a file-location conflict but the file has already been overwritten, losing data `git merge --abort` cannot restore. A direct collision is correctly refused, so the protection is bypassed only for rename-detection destinations; reproduced on versions 2.54.0 through 2.56.0.

### submodule merge reads B's commit from A
A bug report describes merges of a superproject with submodules where Git tries to read from submodule A a commit that exists only in submodule B, reporting "repository corrupt". Patrick Steinhardt confirmed the cause: the delta-base cache retains entries after packfiles are closed, so a pack from another submodule reusing the same `packed_git` address and base offset can return stale cached data; he said he would investigate further.

### refs/reftable: fix on-disk reflog timezone representation
Patrick Steinhardt's v2 series makes the reftable backend store reflog timezones as a signed offset in minutes per the reftable specification instead of the `[+-]HHMM` format, adding `tz_to_minutes()`/`minutes_to_tz()` helpers in `date.c` and converting `refs/reftable-backend.c`, plus a segfault fix in `test-tool dump-reftable -t` and new tests in `t/t0610-reftable-basics.sh`. Today Karthik Nayak reviewed the v2 series and said it looks good to him, and Junio C Hamano stated the series looks really ready and that he will mark the topic for `next`.

### repack: corner cases for cruft-less MIDXs

*Long thread (root 2026/09/30/01-28-34): 14 replies so far from 5 participants over 2 days; 8 messages that day.*

Taylor Blau posted v2 of an eight-patch series fixing corner cases when `repack.midxMustContainCruft=false`: it adds tree/tag roots to the follow walk so the retained MIDX set stays closed under reachability, uses an `oidset` for the extra roots, traverses kept packs during geometric repacks, and fixes `--write-midx=incremental` to include required packs. Elijah Newren objected that the `oidset` conversion can regress pack quality because walk paths affect namehash-based delta selection, presenting examples where subtree-first iteration causes blobs to be written in full instead of delta-compressed.

### filter-branch: fix commit map init from state branch
Grant Moyer reposted v2 fixing `git filter-branch` populating the commit map dir from the state branch with an inverted mapping, correcting it to `from_commit:to_commit` and adding a test that `--state-branch` incremental rewrites use the rewritten parents. Patrick Steinhardt proposed a revised commit message attributing the regression to f6d855091e; Michele Locati corrected the map format description and added his Signed-off-by; Junio C Hamano asked for an explanation of a test change from `--tree-filter "touch file || :"` to `--tree-filter "exit 1"`.

## In brief
- Karthik Nayak's packed-refs `fwrite()` optimization shows ~20% faster ref deletion; Toon Claes listed sanitizations lost by verbatim copying (uppercase OIDs, tab separators, broken peelable refs) and proposed a separate preformatted-entry helper.
- Kazumasa Shigeta's v2 teaches `git stash create` the `-u`/`--include-untracked` and `-a`/`--all` modes via `parse_options()` with `PARSE_OPT_STOP_AT_NON_OPTION`; Junio C Hamano critiqued the commit message for internal mechanics over user-visible implications.
- Julia Evans's series removes `gittutorial-2`; she reported 22 new-user testers of her draft `gittutorial` and prefers shipping it without a branching guide for now.
- Harald Nordgren posted v4 linking CI failure and leak annotations to the test script file and line; Phillip Wood noted the existing annotations are still not escaped.
- Patrick Steinhardt's series adds subcommand groups to `git refs` under "Reference database", "Reading references", and "Writing references"; Junio C Hamano affirmed the `builtin/refs.c` patch with "Nice."
- Phillip Wood defended storing conflict labels in `.git/MERGE_LABELS` over an index extension, fixed CRLF parsing with `strbuf_getline()`, and proposed documenting common repository files in gitrepository as a separate series, which Junio C Hamano agreed to.
- Junio C Hamano posted a maintainer note recording the Git 2.56 release, the roles of the master/maint/next/seen branches, and authoritative external repositories.
