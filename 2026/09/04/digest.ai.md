# Git mailing list daily digest for 2026/09/04

## Long threads

### repack: add --drop-filtered to reclaim space in partial clones

*Long thread (root 2026/07/16/13-28-41): 37 replies so far from 5 participants over 51 days; 1 message that day.*

This GSoC series adds a `--drop-filtered` option to `git repack` so partial clones can reclaim space accumulated from lazily fetched promisor blobs: it enumerates promisor blobs over a size threshold, drops them locally, and relies on the existing lazy-fetch machinery to restore them on demand. Earlier, Samuel Bronson proposed that instead of dying when a blob referenced by the current index would be dropped, the operation should skip it with a warning. Siddharth Shrimali agreed that aborting after the full `enumerate_promisor_blobs()` walk is a poor trade-off, but cautioned that `oidset_remove()` mutates `drop_oids` in place, which would make `--dry-run` stop listing skipped blobs; he proposed collecting index OIDs into a separate skip-set consulted by both the dry-run output and the real drop, and noted the planned drop-log work must not record skipped blobs as dropped.

### hook: introduce the receive-report hook

*Long thread (root 2026/08/18/07-55-55): 37 replies so far from 5 participants over 18 days; 1 message that day.*

This series introduces a `receive-report` hook in `builtin/receive-pack.c` that runs after the reference transaction completes but before the status report is sent to the client: the hook receives the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references as rejected. Karthik Nayak posted v7, rebased on current master, removing the now-redundant `report_v2()`, using a switch statement for the `report_status_version` enum, and dropping an unnecessary curly brace; the fourth patch adds the hook itself, with a non-zero exit producing "receive-report hook failed" for every reference.

### repo: add path keys to git repo info

*Long thread (root 2026/07/16/01-21-31): 30 replies so far from 5 participants over 51 days; 2 messages that day.*

This GSoC series extends `git repo info` with `path.*` keys -- `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup` -- most supporting `.absolute` and `.relative` suffixes. K Jayatheerth proposed queuing the series as-is and moving deduplication of `cdup` and other implementations shared with `git rev-parse` into a follow-up refactoring series. Junio C Hamano identified a bug in `path.superproject-root`: `get_superproject_working_tree()` takes no `repo` parameter and relies on the process working directory, so `git --git-dir=... repo info path.superproject-root.absolute` returns an empty value while `git -C ...` returns the correct root; he stated the function must operate on the repository in question before the series can report that key.

### checkout -m: refine autostash fallback

*Long thread (root 2026/07/25/15-34-27): 26 replies so far from 4 participants over 42 days; 4 messages that day.*

This series makes `git stash` reserve exit status 1 for conflicts (other failures exit 128) and separates the autostash conflict advice from the branch-switch message in `checkout -m`. Today's exchange concerned the CI style job: Harald Nordgren pointed out that `.github/workflows/check-style.yml` runs clang-format but does not catch the mismatched-brace style issue, while Phillip Wood recalled the job was deliberately made to never fail because clang-format output is not guaranteed stable across versions. Junio C Hamano asked whether the `RemoveBracesLLVM: true` setting would have caught it and demonstrated it only removes excess braces; Harald Nordgren agreed version pinning is necessary when linter output is inconsistent.

### doc: cherry-pick: note --no-commit skips CHERRY_PICK_HEAD

*Long thread (root 2026/09/03/12-55-23): 11 replies so far from 4 participants over 2 days; 10 messages that day.*

This documentation change clarifies in `git-cherry-pick.adoc` that `--no-commit` does not record `CHERRY_PICK_HEAD`, so a plain `git commit` afterwards records the committer rather than the original author. Patrick Steinhardt argued against adding tests for every edge case, saying each test needs justification beyond "we didn't have it yet". Phillip Wood confirmed `CHERRY_PICK_HEAD` must not be written under `--no-commit` and suggested a `test_ref_missing` check in the existing conflict-advice test, later disputing that existing coverage exists. Aleksei Sviridkin posted v2 dropping the t3507 test patch and keeping only the documentation clarification; Junio C Hamano agreed a very good justification is needed before adding test overhead.

## The day in brief

Karthik Nayak posted v7 of the receive-report hook series, and Patrick Steinhardt posted an 11-patch series unifying ref storage terminology across `--ref-format`, `GIT_REFERENCE_BACKEND` and `init.defaultRefFormat`. Junio C Hamano found that `path.superproject-root` in the `git repo info` series reads the process working directory instead of the repository, and Siddharth Shrimali refined the `--drop-filtered` index-skip design after Samuel Bronson's RFC.

## Notable threads

### rerere: keep a background gc from killing a rebase

A `git rerere gc` holds `MERGE_RR.lock` while pruning, and auto maintenance after a commit can start one that collides with the sequencer's `repo_rerere()` during a rebase, killing the rebase. The patch makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds, then warn and continue without rerere. Thomas Bachem posted v3 with a distilled commit message, `BUG()` checks on incompatible flag combinations, and documentation that `git rerere`, `git rerere forget` and `git rerere clear` retry and only fail when the time is up; he also identified that `rerere_clear()` on `--abort`/`--skip` must fail rather than leave `MERGE_RR` behind. Junio C Hamano objected to the rationale that a lost recording is nothing next to a rebase that cannot continue, calling a missed preimage recording during a rebase a huge loss.

### Fix inconsistent ref storage format terminology

Patrick Steinhardt posted an 11-patch series renaming `--ref-format=` to `--ref-storage=` across git-init/clone/refs/submodule, `--show-ref-format` to `--show-ref-storage`, `GIT_REFERENCE_BACKEND`/`GIT_DEFAULT_REF_FORMAT` to `GIT_REF_STORAGE`/`GIT_DEFAULT_REF_STORAGE`, and `init.defaultRefFormat` to `init.defaultRefStorage`, keeping old names as hidden aliases and extending `--ref-storage=` to accept URIs with a payload. Karthik Nayak reviewed the series, asking whether a `GIT_DEFAULT_REF_STORAGE_ENVIRONMENT` macro should be added, and said it otherwise looked good. Junio C Hamano objected that unless everything is unified into a single "ref-storage-format", neither "ref-storage" nor "ref-format" is sufficiently clear.

### var: support broken-down idents, default key, multiple args, and -z

This v3 series teaches `git var` to expose individual identity components (`GIT_AUTHOR_NAME`, `GIT_AUTHOR_EMAIL`, `GIT_AUTHOR_DATE` and committer equivalents), `GIT_DEFAULT_KEY` with a `GIT_SIGNING_KEY` alias for the commit signing key, multiple variable arguments, and `-z` NUL-termination. Phillip Wood questioned whether the `GIT_SIGNING_KEY` alias is needed, pointed out that a key can contain `=` making the `var=value` output ambiguous, and proposed that with `-z` the code use the same format as `git config list -z`; he also suggested breaking out of the argument loop at `--` or the first non-option. Junio C Hamano clarified that by "key" Phillip meant "var", endorsed the suggestion, and agreed options should come first per "git help cli".

### dir: fix pathspec prefixes with exclusions

This v3 series fixes pathspec prefix optimization in `dir.c`: exclude items are matched against the full pathname, and the common prefix is found starting with the first non-exclude item. Elijah Newren suggested a deterministic regression test for the negative-pathspec prefix bug, proposing an exclude whose length equals the prefix; Junio C Hamano agreed it was a very good point and demonstrated with `"a/b" ":!x/b" "a/c"` that master wrongly excludes `a/b/m`. Elijah Newren then disputed the description of the preimage, tracing the code to show the old `common_prefix_len()` returns 0 for that input because the `n==0` checks fail for all non-exclude items.

### push: fix --force-if-includes consulting wrong ref

Tyler Cipriani posted a two-patch series fixing `--force-if-includes`, which had been checking the reflog of the local branch named after the destination rather than the ref being pushed, causing false rejections and potential data loss. Patch 1/2 makes `check_if_includes_upstream()` use `ref->peer_ref->name` (resolving `HEAD` to its underlying branch), skips deletions via `is_null_oid`, and rejects pushes from a detached HEAD; patch 2/2 adds a distinct "remote ref unverifiable" rejection advising `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`.

## In brief

- Thomas Bachem posted v2 of "sequencer: leave auto maintenance to the end of a sequence": cherry-pick and revert now also end with one auto maintenance run and suppress it in spawned commands via a new `git_config_append_parameter()` helper.
- Karthik Nayak reviewed patches of the "odb: stop registering in-memory sources" series, confirming the submodule-alternate removal looks in order but calling the cover letter a dense read.
- Aleksei Sviridkin posted v2 of the push `--force-if-includes` fix initializing the reflog timestamp to 0; Junio C Hamano questioned the 0 fallback and said he would rather not have the `Assisted-by` trailer.
- Patrick Steinhardt posted v2 of the "rerere gc" heuristic with `rerere_gc_needed()`; Derrick Stolee said "This version LGTM", and Thomas Bachem noted the lock contention itself is addressed in the separate rerere lock series.
- Junio C Hamano reviewed the `early_scan_options()` series for fast-import, noting it correctly skips option values like the argument after `--depth`, and gave the verdict "Nicely done".
- `jk/rev-info-argv-to-free`, `tc/replay-linearize` and `en/midx-missing-pack-fallback` are now cooking in `next`.
- Ramkumar Ramachandra identified CrowdStrike Falcon scanning as the cause of the git 2.55.0 issues on the high-change-volume LLVM repository and declared the case closed.
