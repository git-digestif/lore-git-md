# Git mailing list daily digest for 2026/09/22

## Long threads

### sequencer: run auto maintenance once at the end of a sequence

*Long thread (root 2026/09/04/07-53-44): 31 replies so far from 5 participants over 19 days; 1 message that day.*

Thomas Bachem's series leaves auto maintenance to the end of a rebase, cherry-pick, or revert sequence: `disable_auto_maintenance()` in `sequencer.c` appends `maintenance.auto=false` to `GIT_CONFIG_PARAMETERS` for the spawned `git commit`, `git merge`, and exec commands, and `run_auto_maintenance()` is called from `run_specific_rebase()` in `builtin/rebase.c` and `run_sequencer()` in `builtin/revert.c` when the sequence completes. Today Bachem detailed why the call sits in the two builtins rather than inside the sequencer: single picks return through `single_pick()`, `continue_single_pick()`, and `sequencer_skip()` without reaching `pick_commits()`, so a sequencer-internal call would need to be placed at the end of `pick_commits()`, at those three returns, and at the cherry-pick and revert aborts (`git rebase --abort` already runs auto maintenance from `finish_rebase()`). He stated that the two builtins are the sequencer's only callers, said both placements work for him, and asked Patrick Steinhardt to say which one to use, offering to reroll accordingly.

## The day in brief

Git v2.56.0-rc2 and Git for Windows 2.56.0-rc2 were announced, the latter dropping Windows 8.1 support. Maciej Ciemborowicz posted v4 of the refs batched-deletion series that reports old OIDs to the reference-transaction hook, and Anupam Mediratta submitted a patch replacing the shell pipeline in `git p4 submit --commit`.

## Notable threads

### refs: report old OIDs for batched deletions

Maciej Ciemborowicz posted v4 of the series teaching `refs_delete_refs()` to accept a parallel array of expected old OIDs so the reference-transaction hook receives real values instead of zero, restoring conditional deletion for branch, tag, and prune operations. The new version always uses `REF_TRANSACTION_ALLOW_FAILURE`, counts individual failures, treats a null old OID as unconditional inside `ref_transaction_delete()`, returns the exact set of failed refs via a new `failed_refs` parameter, and keeps reporting successful fetch/remote prune deletions after a partial failure. Earlier the same day, Junio C Hamano had reviewed v3 arguing for most of those changes and questioning the early `goto cleanup` on partial prune failure, asking whether the successfully deleted refs should still get `[deleted]` reports.

### dir: skip excluded directory with nested repo on prefix match

René Scharfe's patch fixes a corner case in `dir.c` where an ignored directory containing a nested repository was skipped without running pathspec matching. Junio C Hamano reviewed it and found that the `nested_repo` branch in `treat_directory()` could call the new `dir_match()` helper with a NULL `pathspec`, which would segfault because `match_pathspec_with_flags()` dereferences `pathspec` via `GUARD_PATHSPEC`; he proposed wrapping the call in an `if (pathspec)` guard and supplied a test case demonstrating the crash when `git checkout` invokes `verify_clean_subdirectory()` with `pathspec=NULL`.

### reflog: fix default expiry periods

Pushkar Singh posted a patch fixing the reversed default expiry periods in `REFLOG_EXPIRE_OPTIONS_INIT()` in `reflog.h`, restoring 90 days for reachable entries and 30 days for unreachable ones, with a regression test for a 60-day-old reachable entry. Jeff King answered Junio C Hamano's question about the origin of the breakage, stating that the values were flipped by commit 8565827570 ("builtin/reflog: stop storing default reflog expiry dates globally") and that this should go in the commit message.

### git-p4: avoid shell interpretation of commit ids in applyCommit

Anupam Mediratta submitted a patch replacing the shell-based `git diff-tree | git apply` pipeline in `P4Submit.applyCommit()` with a new `diffTreeApply()` helper that uses two argument-vector subprocess calls connected directly through a pipe, removing the shell entirely so a commit id supplied via the unvalidated `--commit` option cannot be interpreted. A regression test in `t/t9803-git-p4-shell-metachars.sh` verifies that shell metacharacters in `--commit` are never executed.

### commit: allow -c/-C for all kinds of --fixup

Erik Cervin-Edin posted v3 of the two-patch series making all message options work with every `--fixup` variation: 1/2 threads `-m` and `-F` through `prepare_to_commit()` for plain, `amend:`, and `reword:` forms, and 2/2 teaches the fixup path to look up the ref given to `-c`/`-C` and seed its message via `prepare_amend_commit()`, removing the incompatibility check entirely since `-C`/`-c` are already rejected by the check grouping them with `-m` and `-F`. He explained that he kept the `prepare_amend_commit()` behavior on purpose so that `git commit --fixup=amend:foo -C foo` and `GIT_EDITOR=: git commit --fixup=amend:foo` produce the same commit.

## In brief

- In the `fetch.shallow` discussion, Phillip Wood recommended fixing `git remote add` in shallow/sparse clones instead of the opt-in config; Junio C Hamano proposed a push-"matching"-style fetch mode keyed off a special refspec; Harald Nordgren suggested always including the upstream's default branch.
- Junio C Hamano questioned the unchecked downcast to the files backend in the repack kept-pack snapshot series, asking whether packfile management should be pushed down to the files backend layer and calling reliance on the downcast "laying mines for our future selves."
- Harald Nordgren posted v2 of the shallow-history advice patch, extending the hint to `git log -<n>` and `git log --since=<date>`; Junio C Hamano retracted his earlier request to advise during the `get_revision()` traversal.
- D. Ben Knoble and Phillip Wood discussed the in-core index merge in `builtin/stash`; Ben Knoble agreed to use `init_ui_merge_options()`, fixed label names, and said he will send v2.
- **fetch.followRemoteHEAD deferred**: Colin Hinton's patch defers `fetch.followRemoteHEAD` validation to `do_fetch()`; Junio C Hamano noted `config_error_nonbool` still triggers during parsing and asked if `remote.*.followRemoteHEAD` in `remote.c` shares it.

- Michał Papis argued for a pre-worktree-remove hook so teardown of external resources (containers, databases) can read the worktree config before removal, noting it applies only to `git worktree remove`, not manual deletion plus `git worktree prune`.
- Johannes Schindelin posted a distilled summary of the Git Contributors' Summit 2026 and asked Junio C Hamano to outline a working v3.0 release plan in the next What's cooking.
- Git v2.56.0-rc2 was announced: 728 non-merge commits since v2.55.0 contributed by 92 people, 36 of them new.
- Git for Windows 2.56.0-rc2 was announced; it drops Windows 8.1 support, so `/ucrt64/bin/git.exe` replaces `/mingw64/bin/git.exe`.
- Johannes Schindelin added a `debian-12` case in `ci/lib.sh` skipping t5559.15 and t5559.16 to work around curl 7.88.1 HTTP/2 stream errors.
- Johannes Schindelin reported GitLab CI Windows job timeouts caused by Runner 19.4.0's `writeClearGitCredentials()` call triggering interactive credential input.
- Phillip Wood said restructuring the `git var` v9 series into three separate patches sounds like a good direction.
