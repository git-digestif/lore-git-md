## Long threads

### `git history squash` subcommand for folding a commit range

*Long thread: 90 replies so far from 9 participants over 73 days.*

Harald Nordgren's series adds `git history squash <revision-range>` to fold a range of commits into one while preserving the oldest commit's authorship and parents. The week centered on hexadecimal case handling in `fixup!` subject matching: Phillip Wood argued that `starts_with()` suffices when the target comes from `git commit --fixup`, but Junio C Hamano countered that the code should not newly accept uppercase hex, aligning with brian m. carlson's broader lowercase-only direction for Git 3.0.

### worktree add message for ambiguous remote branch names

*Long thread: 32 replies so far from 6 participants over 20 days.*

Yoichi Nakayama's series improves the error and advice when `git worktree add` or `git checkout` encounters a branch name matching multiple remote-tracking branches. v8 added a fourth commit making `git worktree add --guess-remote` fail on multiple matches instead of falling back to HEAD; v9 and v10 refined function placement and data types. Junio C Hamano reviewed v10 favorably and stated he will mark it for `next`.

### packfile recovery when a multi-pack-index names a removed pack

*Long thread: 29 replies so far from 5 participants over 13 days.*

Elijah Newren's series fixes objects treated as missing when a geometric repack deletes a pack still named by a stale multi-pack-index. After Jeff King objected that `OBJECT_INFO_QUICK` readers accept false negatives by design, v3 dropped the QUICK-recovery patch, instead fixing `git mktree --batch` to stop passing QUICK and renaming `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return so `find_pack_entry()` recovers on `OBJECT_INFO_SECOND_READ`. Jeff King endorsed the rework, and Junio C Hamano asked whether to mark the series for `next`.

### Path keys for `git repo info`

*Long thread: 28 replies so far from 5 participants over 42 days.*

K Jayatheerth's GSoC series adds path keys such as `path.toplevel`, `path.superproject-root`, and `path.cdup` to `git repo info`. v5 added the `path.cdup` key, but SZEDER Gábor flagged a build break because the series depends on `repo->prefix`, which does not exist on the batch it was queued on; Junio C Hamano acknowledged the issue and said he will requeue the series on top of more recent master.

### Git 3.0: restrict hex object IDs to lowercase only

*Long thread: 25 replies so far from 6 participants over 28 days.*

brian m. carlson's RFC proposes removing uppercase hex from object IDs in Git 3.0, since Git has always emitted lowercase. When Phillip Wood asked for the security problems behind rejecting uppercase, brian cited policy checks against the lowercase form being bypassed by uppercase input and regexps like `[0-9a-f]{40}` failing to match uppercase IDs. Junio C Hamano and Elijah Newren also flagged remaining `HEX_KIND_MIXED` uses in individual patches.

### Converting build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread: 25 replies so far from 5 participants over 24 days.*

D. Ben Knoble's v5 converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration setting, reducing racy-git hits in index-related commands. Junio C Hamano called the patch good but flagged an ordering dependency: `repo_config_values()` must first work on repository instances other than `the_repository` before a `--recurse-submodules` option can operate on a submodule index in the same process.

### receive-report hook for `git-receive-pack`

*Long thread: 21 replies so far from 5 participants over 9 days.*

Karthik Nayak's v4 introduces the `receive-report` hook (renamed from the generic "report"), which runs after the reference transaction but before the report reaches the client; a non-zero exit marks all references rejected. He also reviewed the merge with Justin Tobler's pluggable writes series near the tip of `seen` and reported it looks good.

### commit refusing `--amend` and partial commits during conflict resolution

*Long thread: 21 replies so far from 3 participants over 3 days.*

Elijah Newren's v3 extends the refusal of `git commit --amend` and partial commits (`git commit <paths>`) beyond merge and cherry-pick to `am`, `revert`, and rebase conflict stops, adding preparatory renames and a `sequencer_ongoing_operation()` helper. Phillip Wood approved the logic, while Junio C Hamano questioned the `FROM_REBASE_PICK` naming and suggested `FROM_REBASE_NOW_EMPTY` to distinguish empty commits from picking an empty commit.

### checkout/switch/restore code refactoring

*Long thread: 14 replies so far from 4 participants over 2 days.*

Junio C Hamano posted an eight-patch draft repartitioning `builtin/checkout.c` so that `cmd_switch()` and `cmd_restore()` dispatch directly to `checkout_branch()` or `checkout_paths()` instead of funneling into `checkout_main()`. He flagged it as an early rough draft not planned for `seen`; v2 corrected `post_checkout_hook()` to take a `struct repository *` parameter rather than using `the_repository`.

### `you_still_use_that()` deprecation instructions in usage.c

*Long thread: 8 replies so far from 2 participants over 1 day.*

Junio C Hamano reworded the deprecation text in `usage.c` to "'%s' will be removed soon" and "If you need a replacement:". Michael Montalbo agreed the change makes sense and suggested stressing that the command is not returning.

### builtin command `repo` parameter

*Long thread: 7 replies so far from 3 participants over 1 day.*

Hardik Kumar posted a patch replacing global `the_repository` uses with the previously-UNUSED `repo` parameter in several `cmd_foo()` call sites. Junio C Hamano countered that mixing `the_repository` and `repo` in one function is less consistent than the original, and instead proposed replacing the `struct repository *` parameter of all builtins with `bool has_repo`.

## The period in brief

The week of 2026/08/24 through 2026/08/30 saw seven active days of steady, eventful traffic. Several long-running series reached decision points: Yoichi Nakayama's worktree ambiguous-remote series earned approval for `next`, Elijah Newren's packfile recovery series converged after conceding a QUICK-reader objection, and D. Ben Knoble's runtime `core.useNanosec` patch was called good. Readers should not miss the `git history squash` hex-case debate, brian m. carlson's Git 3.0 lowercase-only RFC, and the use-after-free report in `git stash show`.

## Key developments

### receive-report hook for `git-receive-pack`

Karthik Nayak posted v4 of the series introducing the `receive-report` hook, renamed from "report" after Junio C Hamano objected to the generic name and Patrick Steinhardt proposed "receive-report". The hook runs after the reference transaction but before the client report, reading the pkt-line report on stdin, and a non-zero exit rejects all references with "report hook failed" instead of dying.

### Reftable stack reload avoidance when locked

Karthik Nayak posted v2 of the series avoiding stack reloads inside a locked reference transaction by adding a `locked` bit to `reftable_addition`, so additions on the same stack do not release each other's lock. Benchmarks showed a consistent 15-20% improvement with constant `newfstatat()` counts, and Jeff King reviewed patch 4/4 calling the corrected benchmarks "much nicer".

### trace2: stop allowing die()

Derrick Stolee's v2 introduces a root-level `banned-die.h` included by all trace2 API files, banning die()-capable helpers one at a time so the low-level trace2 API never causes a process-ending failure that could retrigger tracing during die() handling. Elijah Newren flagged a `BANNED(die)` macro mismatch, a `redact_arg()` memcpy dropping the trailing ">", and a bad split between patches; Jeff King noted that banning `die()` catches only direct calls and truly allocation-free trace2 would require a ground-up rewrite.

### send-pack: avoid sending the whole tree when pushing from a shallow clone

Elijah Newren's v2 adds `push.shallowExcludeBoundary` (default true) so pushing from a shallow clone excludes reachable shallow grafts from the pack instead of sending the entire toplevel tree. He argued that defaulting `push.negotiate` would not fix all cases -- rebased pull requests, caches pushed to different repos, explicit `push.negotiate=false`, and protocol v2 unavailable over ssh -- and plans to split further shallow work into two series.

### Worktree lifecycle hooks

Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove`, and `post-worktree-move` hooks, motivated by tools that need reliable triggers when linked worktrees appear, move, or are removed. He argued that a wrapper around Git is insufficient because the component needing notification does not control the callers, and that `git worktree prune` cannot be reproduced externally since only Git knows which entries it actually removed. He proposed consolidating the three hooks into a single `post-worktree` hook with a fixed interface.

### Use-after-free in `git stash show --src-prefix`

Nicolas Le Cam reported that `git stash show --src-prefix=a/ --dst-prefix=b/` prints fragments of freed heap memory instead of the requested prefixes, reproducible on 2.52.0 through 2.54.0. He traced the regression to commit 3ea35c64b, which set `.free_removed_argv_elements = 1` in `show_stash()`; because `OPT_STRING_F` stores pointers into argv elements rather than copying them, freeing those elements leaves `diff_options.a_prefix`/`b_prefix` dangling when the header is emitted.

### Negative pathspecs in `git ls-files` and `git add`

Diogo Castro's patch fixes negative pathspecs in `dir.c` where the common prefix computed from only positive pathspecs is stripped from all pathspecs, causing a negative pathspec shorter than that prefix to degenerate to an empty string that matches everything. Junio C Hamano questioned whether the common prefix should instead be computed across both positive and negative pathspecs, while Diogo objected that extending stripping to negative pathspecs could force unnecessary full working-directory walks.

### Worktree repair: detect relative path in .git file correctly

Yoichi NAKAYAMA's v4 fixes `git worktree repair` failing to convert relative cross-references to absolute paths when `worktree.useRelativePaths=false`, because `read_gitfile_gently()` always resolves to an absolute path. The fix introduces `read_gitfile_raw()` and has `repair_gitfile()` validate with `is_git_directory()`. Junio C Hamano reviewed it as "looks very good to me" and said he will mark it for `next`.

## In brief

**config --global reading both home and XDG files** -- The series makes `git config --global` read both `$HOME/.gitconfig` and the XDG config file, matching what plain `git config` reads; Junio C Hamano acknowledged the `--global` read path was not designed as carefully as the general sequence and that discrepancies are likely bugs.

**Sequencer honoring --empty when a fixup empties its target** -- Farid Zakaria's v4 makes `git rebase --autosquash --empty=drop` drop a commit that a `fixup!` or `squash!` empties, restricting drops to commits that become empty and allowing only the final fixup of a chain to drop the emptied commit.

**branch -d protecting local upstreams** -- Elijah Newren reported a hole in Harald Nordgren's series: with branches A, B, C where B tracks A and C tracks B, `git branch -d A B` protected B but deleted A, leaving surviving B with a missing upstream.

**die_for_incompatible_opts()** -- Junio C Hamano proposed a variadic API taking `<bool, name>` pairs to replace fixed `die_for_incompatible_optN()` wrappers, but later discarded the topic after Jeff King called the varargs interface less safe and René Scharfe proposed alternatives.

**git am --3way postimage blobs** -- Nikita Leshenko proposed recording postimage blobs of cleanly applied patches; Junio C Hamano rejected the eager per-patch overhead in favor of lazy reconstruction when `build_fake_ancestor()` fails.

**odb alternates at creation time** -- Patrick Steinhardt's 8-patch series writes alternates via `odb_source_create_on_disk()` instead of `odb_source_write_alternate()`, fixing a worktree-alternates bug; Toon Claes reviewed placement and Junio C Hamano asked about using the lockfile API for append-mode writes.

**AI-assisted DCO risk** -- Weijie Yuan retracted earlier remarks and reframed the question around AI-assisted DCO risk; Oswald Buddenhagen argued due diligence cannot be assumed with AI tools.

**format-patch --[no-]range-diff-notes** -- Kristoffer Haugsbakk's proposal to tweak notes output in the range diff independently was dropped after Junio C Hamano questioned whether `--no-notes --range-diff-notes` could be defined simply.

**push.negotiate=shallow** -- Patrick Steinhardt proposed the option, enabled by default for shallow clones, as an alternative to new send-pack logic for withholding the shallow boundary tree.

**symbolic-ref exit codes** -- Reviewers disagreed over pinning `git symbolic-ref` exit codes 1 vs 128; Nikolaus Schuetz favored not testing the exact code.

**per-branch stash pop** -- Phillip Wood preferred `git stash pop` options for the latest stash on the current branch over a configurable stash ref; Junio C Hamano cautioned against relying heavily on stash messages.

**git add -e direct apply** -- Junio C Hamano marked for `next` the patch replacing the `run_command()` subprocess in `builtin/add.c` with a direct `apply_all_patches()` call.

**zsh completion for git -C** -- Marked for `next`, the patch skips global options when locating the command.

**git-svn migration noise** -- Wesley Schwengle's v2 silences v1-layout migration noise on plain repositories and adds a `NO_GIT_SVN` Makefile knob.

**CI PR grouping** -- Harald Nordgren's v2 groups pull request workflow runs by PR number with `cancel-in-progress`; Junio C Hamano flagged that `push` and `pull_request` events for the same commit now fall into separate concurrency groups.

**pushcert documentation** -- Lorenz Leutgeb documented `'if-asked'` as a valid value for the `pushcert` option in `Documentation/gitremote-helpers.adoc`.

**Typofixes** -- Hardik Kumar corrected spelling in `versioncmp.c` ("fractionnal" to "fractional"), `git-gui/git-gui.sh`, and `t/t0022-crlf-rename.sh`.

## Looking ahead

Elijah Newren's packfile recovery series and Yoichi Nakayama's worktree ambiguous-remote series both await `next` integration after receiving favorable reviews. The `git history squash` series will likely continue iterating on hex-case handling, influenced by brian m. carlson's Git 3.0 lowercase-only RFC. D. Ben Knoble's `core.useNanosec` series needs the submodule ordering dependency resolved before it can merge, and Domen Kožar's worktree lifecycle hooks proposal will see further discussion on whether to consolidate into a single `post-worktree` hook.