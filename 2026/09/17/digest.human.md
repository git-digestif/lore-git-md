# Git mailing list daily digest for 2026/09/17

## Long threads

### sequencer: leave auto maintenance to the end of a sequence

*Long thread: 30 replies so far from 5 participants over 14 days; 1 message that day.*

Thomas Bachem posted v5 of the three-patch series that moves automatic maintenance out of the sequencer so a rebase, cherry-pick or revert sequence runs `git maintenance run --auto` exactly once when it completes. The code is unchanged from v4; the update adopts Patrick Steinhardt's wording with two corrections (the sequencer itself spawns the `git commit` for a resolved conflict, and the apply backend belongs to `git rebase`), opens 3/3 with Phillip Wood's sentence, and uses Patrick's header comment for `git_config_append_parameter()` plus one sentence on a NULL value. The series adds `git_config_append_parameter()` in `config.c`/`config.h`, calls `run_auto_maintenance()` from `run_specific_rebase()` in `builtin/rebase.c` and `run_sequencer()` in `builtin/revert.c`, and adds `disable_auto_maintenance()` in `sequencer.c`, which appends `maintenance.auto=false` to `GIT_CONFIG_PARAMETERS` for spawned commit, merge and exec commands. Earlier, Patrick had argued that cherry-pick, revert, rebase, `--continue` and `--skip` all reach `pick_commits()` (abort excepted) and preferred calling maintenance when `pick_commits()` finishes and on abort; Thomas explained that single picks never reach `pick_commits()`, so an in-sequencer call would need the end of `pick_commits()` plus those three return paths and the aborts, and he kept the call in the two builtins per Phillip's preference, offering to move it if Patrick prefers.

### advice: scoping hints for the defaultBranchName advice

*Long thread: 25 replies so far from 4 participants over 9 days; 2 messages that day.*

Vsevolod Myalitsin's series makes the `advice.defaultBranchName` hint useful by suggesting the global config scope, since the local-scope command only affects the repository where the advice was already shown while the advice reappears on the next `git init`. Today Jeff King said the advice scoping mechanism patch looks good from a quick read and agreed that passing a pointer to the `advice_setting` to `vadvise()` should go as a separate patch; Vsevolod confirmed the topic reached its intended conclusion and proposed submitting that change separately. Earlier, the discussion had converged on a boolean global hint rather than a full `enum config_scope`, and Junio C Hamano had posted a simpler alternative that just adds `--global` to the squelch message for all advice.

## The day in brief
Thomas Bachem sent v5 of the sequencer auto-maintenance series with wording updates only, Tyler Cipriani posted v6 of the `--force-if-includes` push series that lets fast-forward pushes proceed, and D. Ben Knoble attached a WIP patch fixing the stash autostash duplicate-entry bug.

## Notable threads

### stash autostash leaves a redundant entry with stash.index=true
Eli Barzilay reported that with `stash.index=true`, a successfully applied merge autostash is still stored as a stash entry and deleting `MERGE_AUTOSTASH` fails, requiring a staged change to trigger. Today D. Ben Knoble walked through the stash apply flow, reported that an earlier `reset_tree()` experiment failed six test files, and attached a WIP patch implementing Phillip Wood's `merge_incore_nonrecursive()` approach that passes tests and fixes the bug; Phillip confirmed Ben's reading of the flow and reviewed the WIP patch as looking good, suggesting empty conflict labels, `verbosity=0`, and `oidcpy`.

### push: --force-if-includes should allow fast-forward
Tyler Cipriani posted v6 of the three-patch series checking the pushed ref for `--force-if-includes`. Patch 3/3 defers the reflog rejection until after fast-forward rules are evaluated, so valid fast-forward pushes, including those with expired reflogs, proceed instead of being rejected; patch 1/3 resolves the pushed ref via `ref->peer_ref->name` and rejects non-branch refs that lack a per-branch reflog, and patch 2/3 adds a `pushRefUnverifiable` advice message for that case.

### Coverity fixes from Git for Windows
Johannes Schindelin posted a seven-patch series fixing Coverity issues reported after merging v2.56.0-rc0, covering `writev_in_full()` overflow, length-aware gpg-interface signature matching, incremental MIDX pack ID validation, rerere failure handling, and reftable iterator initialization checks. Junio C Hamano approved patch 4/7, which makes rerere treat preimage/postimage write failures as failures, and suggested dropping the `slen` parameter from `check_signature()` in patch 2/7.

## In brief
- Junio C Hamano observed that the wanted-ref/shallow-info ordering bug in `upload-pack` has existed since the original ref-in-want implementation, so the feature never worked correctly for shallow fetches.
- Mikko Rantalainen reported that `git rerere remaining` skips consecutive conflicted paths that have only a stage-1 index entry, which can make `git mergetool` exit successfully while unresolved conflicts remain.
- Mike Hommey posted v5 moving the Rust gitcore crate into a dedicated `rust/` subdirectory, since a top-level `Cargo.toml` blocks use of the git source as a submodule in a Rust project.
- Brad Smith submitted a patch enabling `RUNTIME_PREFIX` on OpenBSD via `getexecpath()`; Junio C Hamano asked why the new method should replace the one OpenBSD ports already use and which versions it applies to.
- Kaartic Sivaraam proposed the "Implement promisor remote fetch ordering" project for Outreachy's December 2026 cohort and said he will mentor it, possibly with guidance from Christian Couder.
