# Git mailing list daily digest for 2026/08/26

## Long threads

### Ambiguous remote branch messages in worktree add and checkout

*Long thread: 31 replies so far from 6 participants over 19 days; 3 messages that day.*

Yoichi NAKAYAMA posted [PATCH v9 0/4] rerolling the series that improves the error and advice when `git worktree add` or `git checkout` gets a branch name matching multiple remote-tracking branches; the advice lists the matched remotes, and multiple matches with `--guess-remote` are now an error. The only change from v8 is that `advise_disambiguating_remotes()` is defined earlier in the third commit. Earlier, Junio C Hamano had suggested listing the matched remotes and erroring on multiple `--guess-remote` matches. Today he suggested a commit-message body for the first patch and using the better function name from the start, and questioned the `struct string_list **` double indirection in the second patch, suggesting a direct `struct string_list *` would suffice.

### Path keys for git repo info

*Long thread: 28 replies so far from 5 participants over 42 days; 2 messages that day.*

SZEDER Gábor reported that K Jayatheerth's GSoC series adding path keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, `path.cdup`) to `git repo info` depends on `repo->prefix`, which does not exist on the batch it is queued on, leaving non-buildable commits. Junio C Hamano acknowledged the build issue and said he will queue the series on top of more recent master.

### receive-report hook for git-receive-pack

*Long thread: 21 replies so far from 5 participants over 9 days; 3 messages that day.*

Karthik Nayak posted [PATCH v4 0/3] introducing the `receive-report` hook for `git-receive-pack(1)`, renamed from "report" to avoid ambiguity; it runs after the reference transaction but before the report reaches the client, reads the pkt-line report on stdin, and a non-zero exit marks all references rejected. He also reviewed the merge with Justin Tobler's pluggable writes series near the tip of `seen` and reported it looks good. Earlier reviewers had asked for pre-receive-style failure handling and a less generic name.

### you_still_use_that() deprecation instructions in usage.c

*Long thread: 8 replies so far from 2 participants over 1 day; 10 messages that day.*

Junio C Hamano posted a patch rewording the `you_still_use_that()` instructions in `usage.c` -- "If you still use this command, here's what you can do:" becomes "If you need a replacement:" -- and a v2 changing the opening line to "'%s' will be removed soon." Michael Montalbo said the change makes sense, asked whether the double newlines are intentional (Junio confirmed they are), suggested stressing the command is not returning, and agreed "will be removed" sounds great.

## The day in brief
Yoichi NAKAYAMA rerolled the ambiguous-remote-branch message series as v9, Karthik Nayak renamed the new receive-pack hook to `receive-report` in v4, and Junio C Hamano both reworded the `you_still_use_that()` deprecation text and introduced a varargs `die_for_incompatible_opts()` API. SZEDER Gábor also flagged a build break in the `git repo info` path-keys series, which Junio will requeue on newer master.

## Notable threads

### die_for_incompatible_opts() for unbounded option checks
Junio C Hamano posted a two-patch series in `parse-options.c` introducing `die_for_incompatible_opts()`, taking an arbitrary number of `(const char *name, int set)` pairs terminated by NULL; the first patch swaps the argument order of the existing `die_for_incompatible_optN()` functions, and the second reimplements them on the new API, reporting only the first four conflicts to avoid allocation.

### Refusing git commit --amend during conflict resolution
Elijah Newren posted a patch extending the refusal of `git commit --amend` during conflict resolution -- previously only for merge and cherry-pick -- to `am`, `revert`, and `rebase`; for the rebase merge backend it dies only when `stopped-sha` exists without the `amend` marker, so `edit`/`reword`/`break` stops and internal squash amends still work. Phillip Wood judged the logic sound and suggested moving the checks into `sequencer_determine_whence()`; Junio C Hamano suggested using `s->repo` instead of `the_repository` and raised the separate question of rejecting `git commit <paths>` during conflicts.

### Reading both home and XDG files for git config --global
Junio C Hamano reviewed the series making `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` via `config_sequence`, questioning whether forcing forward slashes in `xdg_config_home_for()` on Windows belongs in this topic, suggesting a shorter helper name, arguing `success_count` may be unnecessary, and proposing a dedicated branch for the global scope in `config_with_options()`.

## In brief
- Junio C Hamano marked for `next` the patch replacing the `run_command()` subprocess in `builtin/add.c` with a direct `apply_all_patches()` call for `git add -e`, judging it safe and inviting `add -e` users to interject.
- Junio C Hamano marked for `next` the zsh completion patch supporting `git -C <path>` by skipping global options when locating the command.
- Junio C Hamano declined the patch against leaked abandoned git-header state in `apply`, saying the cross checks are sanity checks and aborting on a discrepancy lets users inspect malformed patches.
- Phillip Wood answered the configurable-stash-ref RFC, preferring `git stash pop` options for the latest stash on the current branch or worktree, plus optional `git stash list` filtering.
- Karthik Nayak defended keeping the lock-state bitfield in `struct reftable_addition` for the reftable stack reload optimization, arguing a move to `reftable_stack` would be more fragile.
