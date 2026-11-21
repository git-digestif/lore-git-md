## Long threads

### `git history squash` subcommand for folding a commit range

*Long thread: 90 replies so far from 9 participants over 73 days.*

Harald Nordgren's series adds `git history squash <revision-range>` in `builtin/history.c`, folding a range of commits into one while preserving the oldest commit's authorship and parents. The week saw continued review of how `fixup!` subject matching should treat hexadecimal case: Phillip Wood argued that `starts_with()` is sufficient when the target comes from `git commit --fixup`, while Junio C Hamano replied that the point is not to add new places accepting uppercase hex, supporting brian m. carlson's `bc/restrict-hex-to-lowercase` direction. The thread remains in active review with no merge decision yet.

### worktree add message for ambiguous remote branch names

*Long thread: 32 replies so far from 6 participants over 20 days.*

Yoichi Nakayama's four-patch series improves the error and advice when `git worktree add` or `git checkout` cannot disambiguate a branch name matching multiple remote-tracking branches, listing the matched remotes and making multiple matches under `--guess-remote` an error instead of falling back to HEAD. After iterating through v8, v9, and v10 -- the last fixing function naming and a `struct string_list **` indirection -- Junio C Hamano reviewed v10, said it looks good, and stated he will mark it for `next` unless other comments arrive.

### packfile recovery when a multi-pack-index names a removed pack

*Long thread: 29 replies so far from 5 participants over 13 days.*

Elijah Newren's series fixes objects being treated as missing when a geometric repack deletes a pack that a stale multi-pack-index still names as owner. V3 drops the earlier QUICK-reader patch after Jeff King argued that `OBJECT_INFO_QUICK` accepts false negatives by design, instead fixing `git mktree --batch` to stop passing QUICK, and renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return so `find_pack_entry()` recovers on `OBJECT_INFO_SECOND_READ`. Jeff King endorsed the decision and reviewed patches 3/4 and 4/4 as looking good; Junio C Hamano asked whether to mark the series for `next`.

### add more path keys to git repo info

*Long thread: 28 replies so far from 5 participants over 42 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info` exposing the working tree root, superproject root, hooks directory, index, grafts file, git prefix, and cdup. V5 added the `path.cdup` key, and Junio C Hamano suggested a common helper shared between `builtin/repo.c` and `builtin/rev-parse.c`. SZEDER Gabor flagged that the series depends on `repo->prefix`, which does not exist on the batch it is queued on, leaving non-buildable commits; Junio acknowledged the build issue and said he will requeue the series on top of more recent master.

### Git 3.0: restrict hex object IDs to lowercase only

*Long thread: 25 replies so far from 6 participants over 28 days.*

brian m. carlson's RFC series proposes removing uppercase hex from object IDs in Git 3.0, since Git has always emitted lowercase and accepting uppercase breaks software assuming a unique hex form. Phillip Wood asked for the security problems behind rejecting uppercase; brian m. carlson gave two cases -- policy checks against the lowercase form can be bypassed by uppercase input, and regexps like `[0-9a-f]{40}` fail to match uppercase IDs -- and said he will fix a dropped case-normalization and a comment typo in v2. Junio C Hamano queued the series while inviting views, and he and Elijah Newren flagged remaining `HEX_KIND_MIXED` uses in individual patches.

### converting build-time USE_NSEC into runtime core.useNanosec

*Long thread: 25 replies so far from 5 participants over 24 days.*

D. Ben Knoble's three-patch series converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration setting so distributors can ship a single binary, reducing racy-git hits in index-related commands. V5 improved the commit message flow in patch 2. Junio C Hamano said the patch looks good to him but flagged an ordering dependency: `repo_config_values()` must first work on repository instances other than `the_repository` before a `--recurse-submodules` option can load and operate on a submodule index in the same process.

### receive-report hook for git-receive-pack

*Long thread: 21 replies so far from 5 participants over 9 days.*

Karthik Nayak's series introduces the `receive-report` hook for `git-receive-pack(1)`, renamed from "report" in v4 to avoid ambiguity. It runs after the reference transaction but before the report reaches the client, reads the pkt-line report on stdin, and a non-zero exit marks all references rejected. He also reviewed the merge with Justin Tobler's pluggable writes series near the tip of `seen` and reported it looks good.

### commit refusing --amend and partial commits during conflict resolution

*Long thread: 21 replies so far from 3 participants over 3 days.*

Elijah Newren's series extends the refusal of `git commit --amend` and partial commits (`git commit <paths>`) beyond merge and cherry-pick to `am`, `revert`, and rebase conflict stops, via a new `sequencer_ongoing_operation()` helper. V3 adds preparatory patches renaming `FROM_REBASE_PICK`/`is_from_rebase()` to reflect empty commits and allowing a partial commit when a rebase pick becomes empty. Phillip Wood approved patch 3/3 and suggested using the sequencer's `rebase_path_stopped_sha()` helpers; Junio C Hamano questioned the `is_from_rebase_empty()` naming, and Newren proposed `FROM_REBASE_NOW_EMPTY`/`is_from_rebase_now_empty()` in response.

## The period in brief

The week of 2026/08/24 through 2026/08/30 saw seven active days of steady, technical traffic dominated by long-running series reaching late-stage review. Readers should not miss Yoichi Nakayama's worktree ambiguous-remote series earning approval for `next`, Elijah Newren's multi-pack-index recovery series converging after Jeff King's review, and brian m. carlson's Git 3.0 lowercase-hex RFC drawing concrete security arguments.

## Key developments

### Reftable stack reload avoidance

Karthik Nayak posted v2 of the series avoiding stack reloads inside a locked reference transaction in the reftable library, adding a `locked` bit to `reftable_addition` so additions on the same stack do not release each other's lock. Benchmarks showed a consistent 15-20% improvement with constant `newfstatat()` counts, and Jeff King reviewed patch 4/4, calling the corrected benchmarks "much nicer" and the idea sound. Nayak also defended keeping the lock-state bitfield in `struct reftable_addition` rather than moving it to `reftable_stack`.

### Config --global reading both home and XDG files

The series makes `git config --global` read both `$HOME/.gitconfig` and the XDG config file, matching what plain `git config` reads. Junio C Hamano explained that reading both per-user files was a deliberate migration aid but acknowledged the `--global` read path was not designed as carefully and that discrepancies with the general sequence are very likely bugs. He reviewed the implementation, questioning whether forcing forward slashes in `xdg_config_home_for()` on Windows belongs in this topic and proposing a dedicated branch for the global scope in `config_with_options()`.

### Trace2 banned-die.h approach

Derrick Stolee's v2 introduces a root-level `banned-die.h` included by all trace2 API files, banning die()-capable helpers one at a time so the low-level trace2 API never causes a process-ending failure that could retrigger tracing during die() handling. Junio C Hamano endorsed the approach and asked whether it could extend to `the_hash_algo` and `the_repository`; Elijah Newren flagged a `BANNED(die)` macro mismatch, a `redact_arg()` memcpy dropping the trailing ">", and a bad split between patches. Jeff King commented that banning `die()` catches only direct calls, and truly allocation-free trace2 would probably require a ground-up no-dependency rewrite.

### Send-pack shallow boundary exclusion

Elijah Newren's v2 adds `push.shallowExcludeBoundary` (default true) so pushing from a shallow clone excludes reachable shallow grafts from the pack instead of sending the entire toplevel tree. He argued that defaulting `push.negotiate` would not fix all cases -- rebased pull requests, caches pushed to different repos, explicit `push.negotiate=false`, and protocol v2 unavailable over ssh -- and plans to split further shallow work into two series. Tests cover the default optimization, the opt-out, and rejection of rootless snapshots by `receive.shallowUpdate` receivers.

### Worktree repair relative-path fix

Yoichi NAKAYAMA's v4 fixes `git worktree repair` failing to convert relative cross-references to absolute paths when `worktree.useRelativePaths=false`, because `read_gitfile_gently()` always resolves to an absolute path. The fix introduces `read_gitfile_raw()` and has `repair_gitfile()` validate with `is_git_directory()`, with tests covering repair from both side and main worktrees. Junio C Hamano reviewed it as "looks very good to me" and said he will mark it for `next` unless further comments arrive.

### Use-after-free in git stash show --src-prefix

Nicolas Le Cam reported that `git stash show --src-prefix=a/ --dst-prefix=b/` prints fragments of freed heap memory instead of the requested prefixes in the diff header, reproducible on 2.52.0 through 2.54.0 but not 2.49.1. He traced the regression to commit 3ea35c64b, which set `.free_removed_argv_elements = 1` in `show_stash()`; because `OPT_STRING_F` stores pointers into argv elements rather than copying them, freeing those elements leaves `diff_options.a_prefix`/`b_prefix` dangling. He warned the same hazard applies to any diff option parsed with `OPT_STRING*` into a `struct diff_options` field.

### Worktree lifecycle hooks

Domen Kozar's v2 series adds `post-worktree-add`, `post-worktree-remove`, and `post-worktree-move` hooks, motivated by tools that need a reliable trigger when linked worktrees appear, move, or are removed. He argued that a wrapper around Git is insufficient because the component needing notification does not control the callers, and that `git worktree prune` cannot be reproduced in a wrapper since only Git knows which entries it actually removed. He proposed consolidating the three hooks into a single `post-worktree` hook with a fixed interface and asked whether that meets the bar for a native hook.

### Negative pathspecs in git ls-files and git add

Diogo Castro's patch fixes a bug in `dir.c` where the common prefix computed from only the positive pathspecs is stripped from all pathspecs, so a negative pathspec shorter than that prefix degenerates to an empty string that matches every file or reads out of bounds. Junio C Hamano questioned whether the common prefix should be computed across both positive and negative pathspecs, while Castro objected to extending the stripping to negative pathspecs, explaining the optimization avoids unnecessary working-directory walks. The discussion remained unresolved at week's end.

## In brief

- **Sequencer ODB-release fix** -- Phillip Wood called the sequencer ODB-release patch a fine immediate fix but suggested `-c gc.auto=false` when forking `git commit`, and both he and Junio C Hamano endorsed studying automatic ODB closure when forking Git commands.
- **push.negotiate=shallow proposal** -- Patrick Steinhardt proposed a `push.negotiate=shallow` option, enabled by default for shallow clones, instead of new send-pack logic for withholding the shallow boundary tree.
- **symbolic-ref exit codes** -- Reviewers disagreed over pinning `git symbolic-ref` exit codes 1 vs 128; Nikolaus Schuetz favored not testing the exact code and offered to sync the adoc NOTES.
- **Per-branch stash pop** -- Phillip Wood proposed a per-branch stash pop option as an alternative to a configurable stash ref; Junio C Hamano cautioned against relying heavily on stash messages.
- **git reset --pull --hard inverse** -- Artur Bieniek proposed the inverse design; Junio C Hamano suggested an alias may be the right feature for this use case.
- **AI DCO discussion** -- Weijie Yuan retracted earlier remarks and reframed the question around AI-assisted DCO risk; Oswald Buddenhagen argued due diligence cannot be assumed with AI tools.
- **branch -d upstream protection hole** -- Elijah Newren reported a hole in Harald Nordgren's series protecting local branches from `git branch -d` when a surviving branch depends on them through an upstream chain; Junio C Hamano had earlier objected that changing `-d` semantics would break workflows.
- **odb alternates at creation** -- Patrick Steinhardt posted an 8-patch series writing alternates at object database creation time via `odb_source_create_on_disk()`; Toon Claes reviewed placement and cleanup tweaks, and Junio asked whether the append-mode write should use the lockfile API.
- **git am --3way postimage** -- Nikita Leshenko proposed recording postimage blobs of cleanly applied patches; Junio rejected the eager per-patch overhead in favor of lazy reconstruction, and Leshenko proposed a simpler lazy alternative.
- **git add -e direct call** -- Junio C Hamano marked for `next` the patch replacing the `run_command()` subprocess in `builtin/add.c` with a direct `apply_all_patches()` call for `git add -e`.
- **zsh completion git -C** -- Junio C Hamano marked for `next` the zsh completion patch supporting `git -C <path>` by skipping global options when locating the command.
- **die_for_incompatible_opts() discarded** -- Junio C Hamano posted v2 of the varargs API but later discarded the topic, saying he did not like the second iteration and liked the preliminary step of the first even less; Jeff King had called the varargs interface less safe, and Rene Scharfe proposed a macro alternative.
- **checkout/switch/restore refactoring** -- Junio C Hamano posted v2 of an eight-patch draft repartitioning `builtin/checkout.c` so `cmd_switch()` and `cmd_restore()` dispatch directly to `checkout_branch()` or `checkout_paths()`, described as an early rough draft that will not be merged to `seen`; D. Ben Knoble affirmed the resulting code organization would be quite lovely.
- **you_still_use_that() rewording** -- Junio C Hamano posted v3 rewording the deprecation text to "'%s' will be removed soon", fixing "mailing list" and correcting two tests.
- **format-patch range-diff-notes dropped** -- Kristoffer Haugsbakk's proposed `--[no-]range-diff-notes` option was dropped after Junio C Hamano questioned whether `--no-notes --range-diff-notes` could be defined simply; Haugsbakk confirmed the use case is unsupported.
- **git-svn v2** -- Wesley Schwengle submitted v2 silencing v1-layout migration noise on plain repositories and adding a `NO_GIT_SVN` Makefile knob.
- **CI PR grouping** -- Harald Nordgren posted v2 grouping pull request workflow runs by PR number with `cancel-in-progress`; Junio flagged that `push` and `pull_request` events for the same commit now fall into separate concurrency groups.
- **Typofixes** -- Hardik Kumar posted NFC patches fixing typos in `versioncmp.c`, `git-gui/git-gui.sh`, and `t/t0022-crlf-rename.sh`; Michael Montalbo reviewed and said the changes LGTM.
- **pushcert documentation** -- Lorenz Leutgeb submitted a one-line patch adding `'if-asked'` as a valid value for the `pushcert` option in `Documentation/gitremote-helpers.adoc`.