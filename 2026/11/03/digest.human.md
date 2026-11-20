# Git mailing list daily digest for 2026/11/03

## Long threads

### path keys for `git repo info`

*Long thread: 52 replies so far from 5 participants over 111 days; 7 messages that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup`, each with absolute and relative suffixes. After Justin Tobler reviewed v7, K Jayatheerth agreed to several changes: exercise the existing `git rev-parse --show-superproject-working-tree` caller in the superproject test instead of the not-yet-introduced `path.superproject-root` key, drop the early returns in the `path.superproject-root` getters, special-case `/dev/null` for `path.hooks` so both formats print it directly (which would also drop the Windows test exclusion), remove the redundant NULL checks in the `path.grafts` getters and probably drop that patch since grafts are deprecated, and fix the `path.cdup` test whose `cd` was not in a subshell. Justin Tobler replied that the `/dev/null` divergence would not block a shared helper between `git rev-parse` and `git repo info`.

### `%gb` placeholder naming the branch of each reflog entry

*Long thread: 32 replies so far from 6 participants over 20 days; 1 message that day.*

Harald Nordgren's v3 series replaces `--operations` and `--historical-branch` with a `%gb` format placeholder that names the branch HEAD was on when each reflog entry was recorded. Phillip Wood reviewed the patch, said "Looks good", suggested rewording "the entries that branch switches, branch renames and rebases leave behind" to "the entries created by ...", criticized "labelled with the branch being rebased" as sounding "very LLM", and accepted the "detached at <commit>" output after trying it. He also reported that with `GIT_REFLOG_ACTION` set during `git rebase`, `%gb` shows incorrect branch names for "sync switch" entries, displaying "sequencer-delete-REBASE_HEAD".

### migrating `loose-object-idx` with batch fsync

*Long thread: 23 replies so far from 4 participants over 13 days; 1 message that day.*

Justin Tobler's series fixes an ODB transaction failure on commit when a compatibility hash is configured, because the temporary-directory migration code did not handle the loose object index file; v3 reads the source mapping before acquiring the lock and adds a comment to `loose_object_idx_merge()`. Karthik Nayak agreed the series is already in good shape after earlier naming suggestions were declined as not worth a reroll.

### root refs vs pseudorefs documentation

*Long thread: 22 replies so far from 4 participants over 13 days; 5 messages that day.*

Andy Koppe's series adds a "root ref" glossary entry, clarifies pseudorefs, and replaces outdated "pseudoref" uses in docs, code and tests. Patrick Steinhardt remained unconvinced by the storage-focused pseudoref definition and proposed an alternative emphasizing different semantics -- pseudorefs can record more than one object ID, may carry extra metadata, and are not handled by the ref backend -- and pointed out a stale MERGE_HEAD quote in the "worktree" entry. Julia Evans suggested describing FETCH_HEAD with a concrete example entry. Junio C Hamano preferred Patrick's definition, suggesting "additional" instead of "different" and "extra" instead of "additional".

### ejecting unreviewed topics from `seen`

*Long thread: 22 replies so far from 5 participants over 100 days; 3 messages that day.*

Patrick Steinhardt proposed rewording "Will merge to 'next'?" to "Will merge to next [on <day>] unless told otherwise" to clarify it for newcomers, and suggested the bar for `seen` be at least one somewhat favorable human review of the overall direction. Junio C Hamano rejected the rewording as "quite a mouthful", said he would keep the '?' and explain it in the document instead, argued that topics entering `seen` without attracting interest need not mesh with other in-flight topics, and said the bar must be a trusted reviewer saying the topic has potential to be a reasonable change, not a drive-by comment.

## The day in brief
A partial-clone regression broke `git merge` in Kubernetes CI, with the first bad commit identified as the pluggable `read_object_info()` change. The early-option-scanning series was marked for `next`, and reviewers converged on the loose-object-idx migration fix and the `%gb` reflog placeholder.

## Notable threads

### lazy fetch in partial clone fails with "not our ref"
Patrick Ohly reported that `git merge` in a blobless partial clone fails with "not our ref" while lazily fetching a missing object, breaking Kubernetes CI PR merges. He bisected v2.52.0 (good) to v2.54.0 (bad) and identified 5946a564 ("odb/source: make read_object_info() pluggable") as the first bad commit, with 2.54.0, 2.56.0 and master failing. The Kubernetes project is reverting to git 2.52.0 to unblock merging. He guessed the new `OBJECT_INFO_SECOND_READ` and reprepare path changes how the promisor lazy fetch is triggered.

### standardizing early option scanning

*Long thread: 20 replies so far from 4 participants over 63 days; 2 messages that day.*

The v4 series introduces an "early-scan" sub-API in parse-options so commands can safely skip option values during hand-rolled early scans. Kaartic Sivaraam reviewed it and said it looks good to him, and Junio C Hamano marked the topic for `next`.

### repository config handling cleanup
Patrick Steinhardt approved Junio C Hamano's patch freezing `struct repo_config_values` with a comment warning against new members until it can support arbitrary repository instances, saying he is very much in favor of merging it so newcomers do not waste time.

## In brief
- Junio C Hamano rejected moving `diff_auto_refresh_index` into `repo_config_values`, since that structure is unusable outside `the_repository` and the variable comes from synthesized non-repository-specific config.
- **Bare --force-with-lease ignored on remote groups**: Patrick Peroša reported a bare `--force-with-lease` is ignored on remote groups: the parent process does not forward the flag to child pushes.

- herr.kaste posted a patch stopping visual roots with columns to their right from being indented in `git log --graph`; Pablo Sabater reviewed the resent patch and said it looks good.
- Ravi Mistry posted v2 of the HTTP exponential-backoff and retry series; Junio C Hamano called it a pleasant read and said patch 3/4 looks good. *Long thread: 8 replies so far from 2 participants over 4 days; 9 messages that day.*
- Kristofer Karlsson's patch teaches the rev-list connectivity check to ignore replace refs; Patrick Steinhardt and Junio C Hamano agreed no extra work is needed to check both original and replacement objects.
- **Junio queues size_t fix for large blobs**: Junio C Hamano queued Elia Pinto's v3 using `size_t` for streamed large blobs in `pack-objects`, fixing size truncation on platforms where `long` is 32 bits but `size_t` is 64 bits.
