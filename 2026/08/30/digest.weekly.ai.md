## Long threads

### `git history squash` subcommand for folding a commit range

*Long thread: 90 replies so far from 9 participants over 73 days.*

Harald Nordgren's series adds `git history squash <revision-range>` to fold a range of commits while preserving the oldest commit's authorship and parents. The week saw continued debate over hexadecimal case handling for `fixup!` markers: Phillip Wood argued `starts_with()` suffices for targets from `git commit --fixup`, while Junio C Hamano countered that the code should not newly accept uppercase hex, pointing to brian m. carlson's lowercase restriction topic. The series remains split into patches for parsing, range validation, branch protection, commit creation, and message editing.

### worktree add message for ambiguous remote branch names

*Long thread: 32 replies so far from 6 participants over 20 days.*

Yoichi Nakayama's four-patch series improves error messages when `git worktree add` or `git checkout` face branch names matching multiple remote-tracking branches. After v8 made multiple `--guess-remote` matches an error, v9 and v10 refined function placement and naming. Junio C Hamano reviewed v10 favorably, stating he will mark it for `next` unless further comments arrive, closing out the 20-day discussion.

### packfile recovery when a multi-pack-index names a removed pack

*Long thread: 29 replies so far from 5 participants over 13 days.*

Elijah Newren's series fixes objects treated as missing when a geometric repack deletes a pack that a stale multi-pack-index still names. After Jeff King objected that `OBJECT_INFO_QUICK` readers accept false negatives by design, v3 dropped the QUICK-recovery patch and instead fixed `git mktree --batch` to stop passing QUICK, renamed `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return, and made `find_pack_entry()` scan for surviving copies on `SECOND_READ`. Jeff King reviewed the final patches favorably, and Junio C Hamano asked whether to mark the series for `next`.

### add more path keys to git repo info

*Long thread: 28 replies so far from 5 participants over 42 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info` exposing the working tree root, superproject root, hooks directory, index, grafts file, git prefix, and cdup. v5 added the `path.cdup` key, and Junio C Hamano suggested a shared helper with `builtin/rev-parse.c`. SZEDER Gabor flagged that the series depends on `repo->prefix`, which does not exist on the batch it was queued on, leaving non-buildable commits; Junio acknowledged and said he will requeue on more recent master.

### Git 3.0: restrict hex object IDs to lowercase only

*Long thread: 25 replies so far from 6 participants over 28 days.*

brian m. carlson's RFC proposes restricting object IDs to lowercase hex in Git 3.0, arguing uppercase input bypasses policy checks against lowercase forms and breaks regexps like `[0-9a-f]{40}`. Junio C Hamano queued the series while inviting views. Phillip Wood asked for the security rationale, which brian provided, and he agreed to fix a dropped case-normalization and comment typo in v2. Junio and Elijah Newren also flagged remaining `HEX_KIND_MIXED` uses in individual patches.

### converting build-time USE_NSEC into runtime core.useNanosec

*Long thread: 25 replies so far from 5 participants over 24 days.*

D. Ben Knoble's series converts the build-time `USE_NSEC` option into the runtime `core.useNanosec` configuration so distributors can ship one binary, reducing racy-git hits in index commands. v5 improved commit message flow. Junio C Hamano said the patch "looks good to me" but flagged an ordering dependency: `repo_config_values()` must work on repository instances other than `the_repository` before a `--recurse-submodules` option can load a submodule index in the same process.

### receive-report hook for git-receive-pack

*Long thread: 21 replies so far from 5 participants over 9 days.*

Karthik Nayak's series introduces the `receive-report` hook for `git-receive-pack(1)`, renamed from the generic "report" after Junio C Hamano and Patrick Steinhardt objected. The hook runs after the reference transaction but before the report reaches the client, reads the pkt-line report on stdin, and a non-zero exit marks all references rejected. v4 incorporated the rename and reviewed well against Justin Tobler's pluggable writes series near the tip of `seen`.

### commit refusing --amend and partial commits during conflict resolution

*Long thread: 21 replies so far from 3 participants over 3 days.*

Elijah Newren's series extends the refusal of `git commit --amend` and partial commits during conflict resolution from merge and cherry-pick to `am`, `revert`, and rebase stops, via a new `sequencer_ongoing_operation()` helper. v3 added preparatory patches renaming `FROM_REBASE_PICK` to reflect empty commits and allowing partial commits when a rebase pick becomes empty. Phillip Wood approved the logic, and Junio C Hamano reviewed patches approvingly while questioning naming.

## The period in brief

The week of 2026/08/24 through 2026/08/30 saw seven active days of heavy traffic, with several long-running threads reaching decision points. Yoichi Nakayama's worktree ambiguous-remote series earned approval for `next`, Elijah Newren's packfile recovery series neared `next` after conceding a QUICK-reader objection, and brian m. carlson defended his Git 3.0 lowercase-hex RFC with concrete security cases. Readers should also note the `git stash show --src-prefix` use-after-free regression and the proposed worktree lifecycle hooks.

## Key developments

### Worktree repair and relative paths

Yoichi NAKAYAMA's v4 fix for `git worktree repair` correctly detects relative paths in the `.git` file when `worktree.useRelativePaths=false` by introducing `read_gitfile_raw()` and validating with `is_git_directory()`. Junio C Hamano reviewed it as "looks very good to me" and said he will mark it for `next` unless further comments arrive.

### Use-after-free in `git stash show --src-prefix`

Nicolas Le Cam reported that `git stash show --src-prefix=a/ --dst-prefix=b/` prints freed heap memory since 2.52.0, traced to commit 3ea35c64b setting `.free_removed_argv_elements = 1` in `show_stash()`, which leaves `diff_options` prefix pointers dangling when `OPT_STRING_F` stores pointers into argv elements. He warned the hazard applies to any diff option parsed with `OPT_STRING*`.

### Worktree lifecycle hooks

Domen Kozar's v2 series proposes `post-worktree-add`, `post-worktree-remove`, and `post-worktree-move` hooks, arguing wrappers cannot reliably detect worktree changes since `post-checkout` does not fire for `--no-checkout` or `--orphan`. He proposed consolidating into a single `post-worktree` hook with a fixed interface issuing events for entries actually pruned.

### Negative pathspecs in `git ls-files` and `git add`

Diogo Castro's patch fixes a bug where the common prefix of positive pathspecs is stripped from all pathspecs, causing negative pathspecs shorter than that prefix to degenerate to empty strings that match everything. Junio C Hamano questioned whether the prefix should be computed across both positive and negative pathspecs, and Diogo defended keeping the optimization for positive-only prefixes to avoid unnecessary directory walks.

### trace2 banned-die.h approach

Derrick Stolee's v2 introduces a root-level `banned-die.h` included by trace2 API files to ban `die()` calls one at a time, preventing process-ending failures that could retrigger tracing during die handling. Elijah Newren flagged a macro mismatch and a `redact_arg()` memcpy issue; Jeff King noted that banning `die()` catches only direct calls and a truly allocation-free trace2 would require a ground-up rewrite.

### send-pack shallow clone optimization

Elijah Newren's v2 adds `push.shallowExcludeBoundary` (default true) so pushing from a shallow clone excludes reachable shallow grafts from the pack instead of sending the entire toplevel tree, which could be gigabytes for large repositories. He argued `push.negotiate` defaults would not cover rebased pull requests or protocol v2 unavailability over ssh.

### reftable stack reload avoidance

Karthik Nayak's v2 avoids stack reloads inside locked reference transactions by adding a `locked` bit to `reftable_addition`, showing 15-20% benchmark improvements and constant `newfstatat()` counts. Jeff King reviewed patch 4/4 calling the corrected benchmarks "much nicer" and the idea sound.

## In brief

**config --global reading both files** -- The series makes `git config --global` read both `$HOME/.gitconfig` and the XDG config file; Junio C Hamano reviewed it, questioning Windows path handling and suggesting a dedicated branch for the global scope.

**Sequencer ODB-release** -- Phillip Wood called the patch a fine immediate fix but suggested `-c gc.auto=false` when forking `git commit`, with both he and Junio C Hamano endorsing study of automatic ODB closure when forking Git commands.

**push.negotiate=shallow** -- Patrick Steinhardt proposed this option, enabled by default for shallow clones, instead of new send-pack logic for withholding the shallow boundary tree.

**format-patch range-diff-notes** -- Kristoffer Haugsbakk's proposal for `--[no-]range-diff-notes` was dropped after Junio C Hamano questioned the interaction with `--notes` and the use case could not be defined simply.

**symbolic-ref exit codes** -- Reviewers disagreed over pinning exit codes 1 vs 128; Nikolaus Schuetz favored not testing the exact code.

**Per-branch stash pop** -- Phillip Wood proposed `git stash pop` options for the latest stash on the current branch as an alternative to a configurable stash ref; Junio C Hamano cautioned against relying heavily on stash messages.

**git reset --pull --hard** -- Artur Bieniek proposed the inverse design; Junio C Hamano suggested an alias may be the right feature.

**AI DCO discussion** -- Weijie Yuan retracted earlier remarks and reframed the question around AI-assisted DCO risk; Oswald Buddenhagen argued due diligence cannot be assumed with AI tools.

**branch -d protecting local upstreams** -- Harald Nordgren proposed protecting branches with surviving upstream dependents; Junio C Hamano objected that changing `-d` semantics would break workflows. Elijah Newren later reported a hole where `git branch -d A B` deleted A while protecting B, leaving B with a missing upstream.

**odb alternates at creation** -- Patrick Steinhardt posted an 8-patch series writing alternates at object database creation time via `odb_source_create_on_disk()`, fixing a worktree-alternates bug; Toon Claes reviewed with placement suggestions.

**git am --3way postimage** -- Nikita Leshenko proposed recording postimage blobs; Junio C Hamano rejected eager per-patch overhead in favor of lazy reconstruction.

**git add -e direct call** -- Junio C Hamano marked for `next` the patch replacing the `run_command()` subprocess in `builtin/add.c` with a direct `apply_all_patches()` call.

**zsh completion git -C** -- Junio C Hamano marked for `next` the patch supporting `git -C <path>` by skipping global options when locating the command.

**sequencer --empty with fixup** -- Farid Zakaria posted v4 making `git rebase --autosquash --empty=drop` drop commits emptied by `fixup!` or `squash!`.

**checkout -m autostash** -- Phillip Wood reviewed Harald Nordgren's refinement, welcoming the conflict-result blank line but pushing back on gating the autostash retry on tracked local changes.

**you_still_use_that rewording** -- Junio C Hamano posted v3 changing the deprecation text to "'%s' will be removed soon."

**git-svn v2** -- Wesley Schwengle submitted v2 silencing v1-layout migration noise and adding a `NO_GIT_SVN` Makefile knob.

**typo fixes** -- Hardik Kumar posted NFC patches fixing "fractionnal" to "fractional" and "similiarity" to "similarity"; Michael Montalbo reviewed LGTM.

**CI grouping PR runs** -- Harald Nordgren posted v2 grouping pull request workflow runs by PR number with `cancel-in-progress`; Junio C Hamano flagged that `push` and `pull_request` events now fall into separate concurrency groups.

**pushcert documentation** -- Lorenz Leutgeb added `'if-asked'` as a valid `pushcert` value in `Documentation/gitremote-helpers.adoc`.

**die_for_incompatible_opts** -- Junio C Hamano discarded the varargs topic after mixed reactions, saying he did not like the second iteration.