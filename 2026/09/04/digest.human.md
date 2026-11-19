# Git mailing list daily digest for 2026/09/04

## Long threads

### repack: add --drop-filtered to reclaim space in partial clones

*Long thread: 37 replies so far from 5 participants over 51 days; 1 message that day.*

This GSoC series adds `--drop-filtered` to `builtin/repack.c` so partial clones can drop promisor blobs above a size threshold and lazily re-fetch them, rebuilding the promisor pack via `repack-promisor.c`. Earlier, Samuel Bronson proposed skipping index-referenced blobs instead of aborting. Siddharth Shrimali agreed today that aborting after the full `enumerate_promisor_blobs()` walk is a poor trade-off, but proposed a separate skip-set so `--dry-run` still lists those blobs, noting the planned drop log must not record them.

### hook: introduce the receive-report hook

*Long thread: 37 replies so far from 5 participants over 18 days; 1 message that day.*

This series adds a `receive-report` hook to `builtin/receive-pack.c` that runs after the ref transaction but before the report reaches the client; its stdout replaces the report and a non-zero exit rejects all references. Karthik Nayak posted v7 today, rebased on master, removing the redundant `report_v2()`, switching on the `report_status_version` enum, and dropping an extra curly brace.

### repo: add path keys to git repo info

*Long thread: 30 replies so far from 5 participants over 51 days; 2 messages that day.*

This GSoC series adds `path.*` keys with absolute/relative suffixes to `git repo info` in `builtin/repo.c`. K Jayatheerth proposed today queuing it as-is and moving deduplication with `git rev-parse` into a follow-up series. Junio C Hamano found that `get_superproject_working_tree()` uses the process working directory rather than the repository, so `git --git-dir=... repo info path.superproject-root.absolute` returns empty; he said it must be corrected before the key can be reported.

### checkout -m: refine autostash fallback

*Long thread: 26 replies so far from 4 participants over 42 days; 4 messages that day.*

This series makes autostash conflicts in `sequencer.c` exit with status 1 (other failures 128) and separates conflict advice from the branch-switch message; v5 was queued earlier. Today's exchange was about style tooling: Harald Nordgren noted the CI style job runs clang-format but misses the mismatched-brace issue, Phillip Wood recalled the job deliberately never fails because clang-format output varies across versions, and Junio C Hamano showed `RemoveBracesLLVM` cannot add braces to a whole if/else cascade.

### doc: cherry-pick: note --no-commit skips CHERRY_PICK_HEAD

*Long thread: 11 replies so far from 4 participants over 2 days; 10 messages that day.*

This documentation-only change to `Documentation/git-cherry-pick.adoc` notes that `--no-commit` skips `CHERRY_PICK_HEAD`, so a plain `git commit` records the committer as author. Aleksei Sviridkin posted v2 today, dropping the t3507 test after Patrick Steinhardt argued tests need stronger justification; Phillip Wood disputed that existing tests cover the conflicting case and urged a `test_ref_missing()` check, while Junio C Hamano sided with Steinhardt on needing justification for new test overhead.

## The day in brief
Karthik Nayak posted v7 of the receive-report hook series, and Tyler Cipriani fixed `--force-if-includes` consulting the wrong ref. Patrick Steinhardt's ref-storage terminology series drew a naming objection, and Siddharth Shrimali refined the `--drop-filtered` index-skip design after Samuel Bronson's RFC.

## Notable threads

### push: fix --force-if-includes consulting the wrong ref
Tyler Cipriani posted a two-patch series fixing `remote.c` and the push/send-pack layer so `--force-if-includes` checks the reflog of the ref being pushed, not a same-named local branch -- a bug behind false rejections and potential data loss. Patch 1/2 resolves `HEAD`, skips deletions, and rejects detached-HEAD pushes; patch 2/2 adds a "remote ref unverifiable" rejection advising `--force-with-lease=<ref>:<expect>`.

### rerere: keep a background gc from killing a rebase
This patch to `rerere.c` makes `setup_rerere()` wait up to `rerere.lockTimeout` for `MERGE_RR.lock` instead of dying, so a background `git rerere gc` cannot kill a rebase. Thomas Bachem posted v3 today with a distilled message and `BUG()` checks on flag combinations. Junio C Hamano objected to the rationale that a lost recording is "nothing next to a rebase that cannot continue", calling a missed preimage during a rebase a huge loss; Bachem also said `rerere_clear()` on `--abort`/`--skip` must fail rather than leave `MERGE_RR` behind.

### Fix inconsistent ref storage format terminology
Patrick Steinhardt posted an 11-patch series renaming `--ref-format=` to `--ref-storage=`, `GIT_REFERENCE_BACKEND` to `GIT_REF_STORAGE`, and `init.defaultRefFormat` to `init.defaultRefStorage`, keeping old names as hidden aliases. Karthik Nayak found it easy to read with small questions. Junio C Hamano objected that neither "ref-storage" nor "ref-format" is clear unless everything becomes "ref-storage-format".

## In brief
- Aleksei Sviridkin posted v2 initializing the reflog timestamp to 0 in `is_reachable_in_reflog()`; Junio C Hamano questioned the 0 fallback and disliked the `Assisted-by` trailer.
- Phillip Wood reviewed the `git var` series (broken-down idents, `GIT_DEFAULT_KEY`, `-z`), questioning the `GIT_SIGNING_KEY` alias and proposing `-z` output match `git config list -z`.
- Elijah Newren proposed a deterministic regression test for the negative-pathspec prefix bug in `dir.c` and disputed Junio C Hamano's reading of the old `common_prefix()` behavior.
- Thomas Bachem posted v2 so rebase, cherry-pick and revert each end with one auto-maintenance run; Junio C Hamano asked whether the cached config value in `replay_ctx` must be a `struct strbuf`.
- Junio C Hamano called the `early_scan_options()` fix for `fast-import --allow-unsafe-features` "Nicely done", noting it skips option values like the argument after `--depth`.
