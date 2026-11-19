# Git mailing list daily digest for 2026/08/26

## Long threads

### Ambiguous remote branch name messages in checkout and worktree add

*Long thread (root 2026/08/08/08-21-41): 31 replies so far from 6 participants over 19 days; 3 messages that day.*

Yoichi Nakayama's series improves the error and advice shown when a branch name matches multiple remote-tracking branches, a case where `git worktree add ../foo-dir bar-topic` previously failed without a meaningful message while `git checkout bar-topic` already explained the ambiguity. The series extracts a shared `advice_disambiguating_remotes()` function in `builtin/checkout.c`, extends `unique_tracking_name()` to collect the matching remote names, makes `git worktree add` die with a descriptive error listing those remotes instead of an "invalid reference" error, and changes `git worktree add --guess-remote` to abort when the path basename matches multiple remotes rather than falling back to HEAD. Today Yoichi Nakayama posted [PATCH v9 0/4], whose only change from v8 is moving `advise_disambiguating_remotes()` earlier in the third commit. Junio C Hamano suggested adding a commit message body to the first patch explaining the motivation, noted that since the second patch renames the function the better name should be used from the beginning, and questioned the double indirection (`struct string_list **`) for the remote-names list, suggesting a direct `struct string_list *` would suffice because the callback only appends to the supplied list.

### New path keys for git repo info

*Long thread (root 2026/07/16/01-21-31): 28 replies so far from 5 participants over 42 days; 2 messages that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info`: `path.toplevel` for the repository working tree root, `path.superproject-root` for the superproject tree from submodules, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` for the offset from the working tree root to the current directory, and `path.cdup` for the relative path back to the top level, with absolute and relative suffixes where applicable. Today SZEDER Gábor reported that the series depends on `repo->prefix` but is queued on top of a batch where that field does not exist, producing non-buildable commits, and noted the field was introduced in a later merge. Junio C Hamano acknowledged the build issue and said he will queue the series on top of more recent master.

### receive-report hook for git-receive-pack

*Long thread (root 2026/08/18/07-55-55): 21 replies so far from 5 participants over 9 days; 3 messages that day.*

Karthik Nayak's series introduces a `receive-report` hook for `git-receive-pack(1)` that runs after the reference transaction completes but before the report is sent to the client; the hook receives the pkt-line encoded report on stdin, its stdout replaces the report sent to the user, and a non-zero exit marks all references as rejected. Today Karthik Nayak reviewed the merge of the series with Justin Tobler's pluggable writes series near the tip of `seen` and reported that it looks good, and posted [PATCH v4 0/3], renaming the hook from "report" to "receive-report" to avoid ambiguity.

### Reworded you_still_use_that() instructions

*Long thread (root 2026/08/26/14-33-31): 8 replies so far from 2 participants over 1 day; 10 messages that day.*

Junio C Hamano's patch rewords the instructions printed by `you_still_use_that()` in `usage.c` when a command nominated for removal is used. The message previously opened with "If you still use this command, here's what you can do:" and listed the BreakingChanges docs, the mailing list, and an email address; it now says "If you need a replacement:" with reworded bullets, and [PATCH v2] strengthens the opening line from "nominated for removal" to "will be removed soon". Today Michael Montalbo said the change makes sense but suggested the final bullet reinforce that the command is not coming back, asked whether the new double newlines are intentional (Junio C Hamano confirmed they are), and accepted the reasoning for not repeating "replacement" in the final bullet while reiterating his main feedback.

## The day in brief

Yoichi Nakayama posted v9 of the series improving ambiguous-remote-branch messages for `git worktree add` and `git checkout`, and Karthik Nayak posted v4 of the `receive-report` hook after confirming its merge with the pluggable writes series looks good. SZEDER Gábor flagged that K Jayatheerth's `git repo info` path-key series does not build on the base it was queued on, and Elijah Newren extended the refusal of `git commit --amend` during conflict resolution to `am`, `revert`, and `rebase`.

## Notable threads

### config: read both home and XDG files for --global

This series fixes the inconsistency where `git config list --global` and `git config get --global` read only `$HOME/.gitconfig` while plain `git config` reads both that file and `$XDG_CONFIG_HOME/git/config`. Today Junio C Hamano questioned whether forcing forward slashes in `xdg_config_home_for()` on Windows must be part of this fix, asking where the backslashes come from and noting the log message does not explain why; he corrected the claim that command output is irrelevant when no global config exists, suggested renaming `attempt_git_config_from_file_with_options()` to something shorter like `try_config()`, expressed doubt that `success_count` is needed until later steps are seen, and argued for teaching `git_config_system()` to take `config_options` or adding a dedicated branch for the global case in `config_with_options()`.

### commit: refuse to amend during conflict resolution

Elijah Newren's patch extends the long-standing refusal of `git commit --amend` during conflict resolution, previously limited to merge and cherry-pick, to `am`, `revert`, and `rebase`; for the rebase merge backend it dies only when `stopped-sha` exists without the `amend` marker, so clean `edit`/`reword`/`break` stops and the sequencer's internal amends remain allowed. Today Phillip Wood judged the logic sound, noting a conflicted fixup should still permit amending, and suggested moving the checks into `sequencer_determine_whence()`; Junio C Hamano called the stopped-sha/amend reasoning sound, raised the separate question of also rejecting `git commit <paths>` during conflict resolution, and suggested using `s->repo` instead of `the_repository`.

### apply: abandoned git-header state

This patch addressed `find_header()` in the apply machinery leaving partially parsed git-header state behind when a "diff --git" line is abandoned for a later traditional header, which could combine incompatible metadata. Today Junio C Hamano decided not to apply the patch, explaining that the cross checks are primarily sanity checks and that having the parser notice a discrepancy and abort is a good thing so the user can inspect and fix a malformed patch.

### zsh completion after git -C

Lutz Lengemann's patch teaches the zsh completion wrapper to handle the global `-C` option, which previously left `git -C <path> <command> <TAB>` with nothing to complete, by adding `-C` to the specification, locating the command by skipping global options and their arguments, and collecting `-C` paths so helpers resolve the right repository. Today Junio C Hamano marked the topic for `next`.

### git add -e without a subprocess

Gatla Vishweshwar Reddy's v4 patch replaces the `run_command()` call that spawns `git apply --recount --cached` in `builtin/add.c` with a direct `apply_all_patches()` call using an initialized apply state, avoiding the fork and keeping the operation in-process. Today Junio C Hamano marked the patch for `next`, judging it safe and correct, and invited `add -e` users to interject with concerns.

## In brief

- Junio C Hamano posted a two-patch series introducing `die_for_incompatible_opts()`, which accepts an unbounded number of mutually incompatible options via `(const char *name, int set)` pairs terminated by NULL; the first patch swaps the argument order of the existing `die_for_incompatible_optN()` functions, and the new function deliberately reports only the first four conflicts to avoid allocation costs.
- Harald Nordgren's patch groups pull-request CI workflow runs by PR number and cancels superseded runs while keeping push events grouped by commit hash; Junio C Hamano analyzed the concurrency change, flagged an unverified concern that simultaneous workflows sharing a group might leave some runs unable to start, and noted non-PR events are unaffected.
- Taylor Blau replied to Junio C Hamano's question about the `midx-write` custom-base filtering comment, saying nothing had happened since his earlier comment; instead of sending a prepared reroll he shared a proposed replacement comment in `midx-write.c` and asked whether it was clear.
- Phillip Wood responded to the RFC on a configurable stash ref, proposing options on `git stash pop` to pop the most recent stash created on the current branch or in the current worktree, and suggesting `git stash list` filtering by branch or worktree as a separate addition.
- Karthik Nayak agreed with Patrick Steinhardt that tracking lock state via a bitfield in `struct reftable_addition` is fragile, but explained moving it to `reftable_stack` would be more fragile, concluding the current approach is acceptable.
- Junio C Hamano noted the `commit --fixup` with `-c`/`-C` topic had hung without reviews and asked for takers, and pointed out the `die_for_incompatible_opt3` check blocking `-C`, `-c`, and `-F` together is redundant because the later `die_for_incompatible_opt4` already covers those combinations.
- Harald Nordgren confirmed a passage from Junio C Hamano asking whether `git branch -d` needed the same protection as delete-merged was what he remembered; Junio C Hamano said he had more likely misstated and appreciated the new machinery for an improved protection feature.
