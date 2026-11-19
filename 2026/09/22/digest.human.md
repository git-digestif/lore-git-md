# Git mailing list daily digest for 2026/09/22

## Long threads

### sequencer: run auto maintenance once at the end of a rebase, cherry-pick or revert sequence

*Long thread: 31 replies so far from 5 participants over 19 days; 1 message that day.*

Thomas Bachem's series makes merge-backend sequences run `git maintenance --auto` once when the sequence completes instead of inside the `git commit`, `git merge` and exec commands the sequencer spawns: `disable_auto_maintenance()` in `sequencer.c` appends `maintenance.auto=false` to `GIT_CONFIG_PARAMETERS` for those children, and `run_auto_maintenance()` is called from `run_specific_rebase()` in `builtin/rebase.c` and `run_sequencer()` in `builtin/revert.c`. Today he spelled out why the call sits in the two builtins: single picks return through `single_pick()`, `continue_single_pick()` and `sequencer_skip()` without reaching `pick_commits()`, so a sequencer-internal call would need to sit at the end of `pick_commits()`, at those three returns, and at the cherry-pick and revert aborts (`git rebase --abort` already runs maintenance from `finish_rebase()`), while the two builtins are the sequencer's only callers. He said both placements work for him and asked Patrick Steinhardt to say which one to use, offering to reroll accordingly.

## The day in brief

Maciej Ciemborowicz posted v4 of the refs batched-deletion series addressing Junio C Hamano's review comments, Anupam Mediratta submitted a git-p4 fix that removes shell interpretation of `--commit` values, and Thomas Bachem asked Patrick Steinhardt to choose between placing the sequencer's auto-maintenance call in the builtins or inside the sequencer itself.

## Notable threads

### git-p4: avoid shell interpretation of commit ids in applyCommit

Anupam Mediratta submitted a patch replacing the shell-based `git diff-tree | git apply` pipeline in `P4Submit.applyCommit()` with a new `diffTreeApply()` helper that uses two argument-vector subprocess calls connected directly through a pipe. The commit id can come verbatim from the unvalidated `--commit` option, so a value like `$(some-command)` was previously executed by the shell; the change removes the shell entirely. A regression test in `t/t9803-git-p4-shell-metachars.sh` verifies that shell metacharacters in `--commit` are never executed.

### refs: report old OIDs for batched deletions

Maciej Ciemborowicz posted v3 and then v4 of a series teaching `refs_delete_refs()` to accept a parallel array of expected old OIDs so the reference-transaction hook receives real values instead of zero for branch, tag, fetch and remote deletions. v4, rebased onto Git 2.56-rc2, always uses `REF_TRANSACTION_ALLOW_FAILURE`, counts individual failures, treats a null old OID as unconditional inside `ref_transaction_delete()`, returns the exact set of failed refs, and continues reporting successful fetch/remote prune deletions after a partial failure. Junio C Hamano reviewed v3, arguing the flag should always be passed, failures counted rather than set to 1, and that partial prune failures should still report the successfully deleted refs; he later said he does not recall offering to prepare a patch.

### repack: don't lose objects to a ".keep" that appears mid-run

This series fixes a race where a concurrent push or fetch installs a `.keep` next to a pack that `git repack -d` has already decided to delete, causing pack-objects to omit the pack's objects while repack deletes the pack anyway. The fix hands pack-objects the snapshot repack took at startup. Junio C Hamano reviewed v2 2/5 and questioned whether the unchecked downcast to the files backend is safe long-term, asking whether packfile management should be pushed down to the files backend layer; he noted that until another ODB backend materializes the change might be sufficient, but relying on the unchecked downcast feels like "laying mines for our future selves."

## In brief

- Pushkar Singh fixed the reversed default reflog expiry periods in `REFLOG_EXPIRE_OPTIONS_INIT()` (90 days for reachable, 30 for unreachable) with a regression test; Jeff King traced the flip to commit 8565827570 and said that belongs in the commit message.
- Erik Cervin-Edin posted v3 allowing `-m`/`-F`/`-c`/`-C` for all `--fixup` variations, removing the `-C`/`-c`/`--fixup` incompatibility check entirely.
- Junio C Hamano reviewed René Scharfe's `dir.c` patch for excluded directories containing a nested repo and found the new `dir_match()` helper would segfault on a NULL pathspec, proposing an `if (pathspec)` guard.
- Discussion of Harald Nordgren's `fetch.shallow` config: Phillip Wood recommended fixing `git remote add` instead, Junio C Hamano proposed a fetch mode that fetches each local branch's upstream, and Harald Nordgren suggested always including the upstream's default branch.
- Harald Nordgren posted v2 of the shallow-history advice hint; Junio C Hamano retracted his earlier request to advise during the `get_revision()` traversal.
- Maciej Ciemborowicz acknowledged that making copy/rename a whole-transaction property does not compose and said he will rework it so the state belongs to individual updates.
- D. Ben Knoble and Phillip Wood discussed the in-core index merge in `builtin/stash`; Ben queued the `init_ui_merge_options()` change and said he will send v2.
