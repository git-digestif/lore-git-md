# Git mailing list daily digest for 2026/09/17

## Long threads

### sequencer: leave auto maintenance to the end of a sequence

*Long thread (root 2026/09/04/07-53-44): 30 replies so far from 5 participants over 14 days; 1 message that day.*

Thomas Bachem posted v5 of the three-patch series that moves automatic maintenance out of the sequencer and runs it once when a rebase, cherry-pick, or revert sequence completes. The code is unchanged from v4; the revision adopts Patrick Steinhardt's wording in 2/3 with two corrections (the sequencer itself spawns the `git commit` for a resolved conflict, and the apply backend belongs to `git rebase`), opens 3/3 with Phillip Wood's sentence, and uses Patrick's header comment for `git_config_append_parameter()` plus one sentence on a NULL value. The series adds `git_config_append_parameter()` in `config.c`/`config.h`, calls `run_auto_maintenance()` from `run_specific_rebase()` in `builtin/rebase.c` and `run_sequencer()` in `builtin/revert.c`, and adds `disable_auto_maintenance()` in `sequencer.c`, which appends `maintenance.auto=false` to `GIT_CONFIG_PARAMETERS` for spawned `git commit`, `git merge`, and exec commands so background maintenance cannot race the sequence. Earlier, Thomas explained that single picks never reach `pick_commits()`, so an in-sequencer call would need changes at several entry points; he kept the call in the two builtins per Phillip's preference and offered to move it into the sequencer if Patrick prefers.

### advice: configuration scope hint for the defaultBranchName advice

*Long thread (root 2027/08/29/00-49-58): 25 replies so far from 4 participants over 9 days; 2 messages that day.*

Jeff King said the advice scoping mechanism patch looks good from a quick read and agreed that passing a pointer to the `advice_setting` to `vadvise()` should be submitted as a separate patch. Vsevolod Myalitsin confirmed the topic was at its intended conclusion and proposed submitting that change separately. The underlying series, "defaultBranchName advice is useless", addresses the fact that the advice for setting the default branch name suggests `git config set advice.defaultBranchName false`, which writes to the local repository even though the advice appears during `git init` and reappears for the next repository; the series adds a configuration scope hint to advice settings and suggests the global scope for that advice.

## The day in brief

Thomas Bachem posted v5 of the sequencer auto-maintenance series with unchanged code and refined commit messages, and Vsevolod Myalitsin's advice-scoping work reached its intended conclusion. D. Ben Knoble attached a WIP patch that fixes the stash autostash duplicate-entry bug, and Tyler Cipriani posted v6 of the `--force-if-includes` push series allowing fast-forwards.

## Notable threads

### stash autostash leaves a redundant stash entry with stash.index=true

The thread concerns a bug where an autostash applied successfully during a fast-forward merge is still stored as a stash entry and deleting `MERGE_AUTOSTASH` fails, requiring a staged change to trigger. Phillip Wood confirmed that the stash apply flow's update-then-reset-then-update of the index cannot be avoided if the code is to error out on conflicts when merging the index. D. Ben Knoble walked through the flow, reported that an earlier `reset_tree(&c_tree, 0, 1)` experiment failed 6 test files and did not fix the bug, and attached a WIP patch implementing Phillip's `merge_incore_nonrecursive()` approach that passes tests and fixes the bug. Phillip confirmed Ben's reading of the flow and reviewed the WIP patch as looking good, suggesting simplifications such as empty conflict labels, `verbosity=0`, and `oidcpy` for copying the merged tree.

### push: check pushed ref for --force-if-includes

Tyler Cipriani posted v6 of the three-patch series. Patch 1/3 changes `check_if_includes_upstream` to resolve the pushed ref via `ref->peer_ref->name`, explicitly rejecting pushes of non-branch refs that lack a per-branch reflog while allowing deletions. Patch 2/3 adds a `pushRefUnverifiable` advice message for rejections where the source is not a branch. Patch 3/3 defers the `--force-if-includes` reflog rejection until after fast-forward rules are evaluated, so valid fast-forward pushes, including those with expired reflogs, proceed instead of being rejected.

### move the rust gitcore crate into a dedicated subdirectory

Mike Hommey posted v5 moving all Rust-specific files into a `rust/` subdirectory. A top-level `Cargo.toml` implies that `cargo build` works directly, which produces nothing useful, and it prevents the git source from being included when used as a submodule of a Rust project because cargo skips directories containing a `Cargo.toml`.

### upload-pack: swap wanted-ref/shallow-info responses

Royce Remer's patch swaps the send order in `upload_pack_v2()` so shallow-info precedes wanted-ref info, fixing protocol v2 shallow fetches against servers with `uploadpack.allowRefInWant` configured, which otherwise exit with "expected 'packfile', received 'shallow-info'". Junio C Hamano observed that the ordering bug has existed since the original ref-in-want implementation in 2018 because the on-the-wire protocol never changed, so the feature never worked correctly for shallow fetches.

### rerere remaining skips consecutive conflicted paths

Mikko Rantalainen reported that `git rerere remaining` skips consecutive conflicted paths when each path has only a stage-1 index entry, causing `git mergetool` to exit with status 0 while unresolved conflicts remain. He identified the likely cause as a loop in `check_one_conflict()` in `rerere.c` that advances past stage-1 entries without verifying they belong to the same pathname, suggested a `ce_same_name()` check, and provided a minimized reproducer along with the `git mergetool -- .` workaround.

## In brief

- Johannes Schindelin posted a 7-patch series fixing Coverity issues reported by Git for Windows after v2.56.0-rc0, covering writev overflow, gpg-interface signature matching, MIDX pack ID validation, and rerere failure handling; Junio C Hamano approved the rerere patch.
- **OpenBSD getexecpath patch**: Brad Smith proposed RUNTIME_PREFIX on OpenBSD using getexecpath(); Junio C Hamano asked why use it over the method ports already use, whether the old way is being removed, and which versions support it.

- Kaartic Sivaraam proposed the "Implement promisor remote fetch ordering" project for Outreachy's December 2026 cohort and said he will mentor it, possibly with guidance from Christian Couder.
- Brigham Campbell offered to switch `git-contacts` stdin input to a `-` argument if Junio C Hamano prefers that interface.
- Tuomas Ahola suggested amending the v2.56.0-rc1 announcement entry about `git history` segfaulting in a corrupt repository to note it has been corrected.
- SZEDER Gábor reported that precompiling `git-compat-util.h` works with Clang and probably other LLVM-based compilers.
- Johannes Schindelin announced Git for Windows 2.56.0-rc1, which drops Windows 8.1 support, moves `git.exe` to `/ucrt64/bin/`, and fixes the external OpenSSH installer option broken in v2.55.0(5).
- Mark C. Chu-Carroll posted v3 updating `t/t4001-diff-rename.sh`, `t/t4009-diff-rename-4.sh`, and `t/t4010-diff-pathspec.sh` to modern test style.
- Yoichi NAKAYAMA submitted a patch adding his canonical entry to `.mailmap`.
