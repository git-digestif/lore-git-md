## Long threads

### `git history squash` subcommand for folding a commit range
*Long thread: 90 replies so far from 9 participants over 73 days.*
Harald Nordgren's series adds `git history squash <revision-range>` to fold a range of commits into one while preserving the oldest commit's authorship and parents. The week saw continued review of how `fixup!` subject matching should treat hexadecimal case: Phillip Wood argued that `starts_with()` is sufficient when the target comes from `git commit --fixup`, while Junio C Hamano countered that the point is to avoid adding new places accepting uppercase hex, supporting brian m. carlson's broader effort to restrict object IDs to lowercase. The series remains in review with no merge decision yet.

### worktree add message for ambiguous remote branch names
*Long thread: 32 replies so far from 6 participants over 20 days.*
Yoichi Nakayama's four-patch series improves the error and advice when `git worktree add` or `git checkout` encounters a branch name matching multiple remote-tracking branches. The series progressed from v8 through v10 during the week, adding a commit that makes `git worktree add --guess-remote` fail on multiple matches instead of falling back to HEAD. Junio C Hamano reviewed v10, called it good, and stated he will mark it for `next` unless further comments arrive.

### packfile recovery when a multi-pack-index names a removed pack
*Long thread: 29 replies so far from 5 participants over 13 days.*
Elijah Newren's series fixes objects being treated as missing when a geometric repack deletes a pack that a stale multi-pack-index still names. After Jeff King objected that `OBJECT_INFO_QUICK` is designed to accept false negatives, Newren dropped the QUICK-reader patch in v3 and instead fixed `git mktree --batch` to stop passing `OBJECT_INFO_QUICK`. The series now renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return so `find_pack_entry()` can recover on `OBJECT_INFO_SECOND_READ`. Jeff King endorsed the v3 approach, and Junio C Hamano asked whether to mark the series for `next`.

### add more path keys to git repo info
*Long thread: 28 replies so far from 5 participants over 42 days.*
K Jayatheerth's GSoC series adds path keys to `git repo info` exposing the working tree root, superproject root, hooks directory, index, grafts file, git prefix, and cdup. v5 added the `path.cdup` key, and Junio C Hamano suggested a common helper library shared between `builtin/repo.c` and `builtin/rev-parse.c`. SZEDER Gábor flagged that the series depends on `repo->prefix`, which does not exist on the batch it is queued on, leaving non-buildable commits; Junio acknowledged the build issue and said he will requeue the series on more recent master.

### converting build-time USE_NSEC into runtime core.useNanosec
*Long thread: 25 replies so far from 5 participants over 24 days.*
D. Ben Knoble's three-patch series converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration setting, reducing racy-git hits in index-related commands. v5 improved the commit message flow. Junio C Hamano said the patch looks good but flagged an ordering dependency: `repo_config_values()` must first work on repository instances other than `the_repository` before a `--recurse-submodules` option can load a submodule index in the same process.

### Git 3.0: restrict hex object IDs to lowercase only
*Long thread: 25 replies so far from 6 participants over 28 days.*
brian m. carlson's RFC series proposes removing uppercase hex from object IDs in Git 3.0. During the week, brian defended the restriction with two security cases: policy checks against the lowercase form can be bypassed by uppercase input, and regexps like `[0-9a-f]{40}` fail to match uppercase IDs. Junio C Hamano queued the series while inviting views, and Phillip Wood, Elijah Newren, and others flagged remaining `HEX_KIND_MIXED` uses and test questions in individual patches.

### receive-report hook for git-receive-pack
*Long thread: 21 replies so far from 5 participants over 9 days.*
Karthik Nayak's series introduces a `receive-report` hook for `git-receive-pack(1)`, renamed from the generic "report" in v4 to avoid ambiguity. The hook runs after the reference transaction but before the report reaches the client, reads the pkt-line report on stdin, and a non-zero exit marks all references rejected. Nayak also reviewed the merge with Justin Tobler's pluggable writes series and reported it looks good.

### commit refusing --amend and partial commits during conflict resolution
*Long thread: 21 replies so far from 3 participants over 3 days.*
Elijah Newren's series extends the refusal of `git commit --amend` and partial commits (`git commit <paths>`) beyond merge and cherry-pick to `am`, `revert`, and rebase conflict stops, via a new `sequencer_ongoing_operation()` helper. v3 added preparatory patches renaming `FROM_REBASE_PICK` to reflect empty commits and allowing partial commits when a rebase pick becomes empty. Phillip Wood approved patch 3/3, and Junio C Hamano reviewed several patches approvingly while questioning naming choices.

### checkout/switch/restore code refactoring
*Long thread: 14 replies so far from 4 participants over 2 days.*
Junio C Hamano posted an eight-patch draft series repartitioning the monolithic `checkout_main()` in `builtin/checkout.c` so that `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` dispatch directly to `checkout_branch()` or `checkout_paths()`. He described it as an early rough draft he does not plan to merge even to `seen`, noting that patch 8/8 illustrates libification by having `post_checkout_hook()` take a `struct repository *` parameter. D. Ben Knoble affirmed the resulting organization would be lovely.

### you_still_use_that() deprecation instructions in usage.c
*Long thread: 8 replies so far from 2 participants over 1 day.*
Junio C Hamano reworded the `you_still_use_that()` instructions in `usage.c`, changing "If you still use this command, here's what you can do:" to "If you need a replacement:" and later to "'%s' will be removed soon." Michael Montalbo reviewed the changes, confirmed the double newlines are intentional, and agreed the "will be removed" phrasing sounds great.

### builtin command `repo` parameter
*Long thread: 7 replies so far from 3 participants over 1 day.*
Hardik Kumar posted a patch replacing global `the_repository` uses with the previously-UNUSED `repo` parameter in several `cmd_foo()` builtins. Junio C Hamano countered that mixing `the_repository` and `repo` in one function is less consistent, and instead posted an illustrative patch replacing the `struct repository *` parameter of all builtins with `bool has_repo`, asking for opinions on the direction.

## The period in brief
The week of 2026/08/24 through 2026/08/30 saw heavy traffic across seven active days, with several long-running series reaching decision points. Yoichi Nakayama's worktree ambiguous-remote series earned approval for `next`, Elijah Newren's packfile recovery series won Jeff King's endorsement, and D. Ben Knoble's runtime `core.useNanosec` patch was called good by Junio C Hamano. Readers should not miss the worktree repair fix also marked for `next`, the stash show use-after-free regression report, and the ongoing Git 3.0 lowercase-hex RFC.

## Key developments

### trace2: stop allowing die()
Derrick Stolee's v2 introduces a root-level `banned-die.h` included by all trace2 API files, banning die()-capable helpers one at a time so the low-level trace2 API never causes a process-ending failure that could retrigger tracing during die() handling. Junio C Hamano endorsed the approach and asked whether it could extend to `the_hash_algo` and `the_repository`. Elijah Newren flagged a `BANNED(die)` macro mismatch and a `redact_arg()` memcpy issue, while Jeff King noted that banning `die()` catches only direct calls and truly allocation-free trace2 would require a ground-up rewrite.

### send-pack: avoid sending the whole tree when pushing from a shallow clone
Elijah Newren's v2 adds `push.shallowExcludeBoundary` (default true) so pushing from a shallow clone excludes reachable shallow grafts from the pack instead of sending the entire toplevel tree, which for large repositories could be gigabytes instead of kilobytes. He argued that defaulting `push.negotiate` would not fix all cases, and plans to split further shallow work into two series. Tests cover the default optimization, the opt-out, and rejection of rootless snapshots by `receive.shallowUpdate` receivers.

### config --global reading both home and XDG files
The series makes `git config --global` read both `$HOME/.gitconfig` and the XDG config file, matching what plain `git config` reads. Junio C Hamano explained that reading both per-user files was a deliberate migration aid but acknowledged the `--global` read path was not designed as carefully and that discrepancies are likely bugs. He reviewed the implementation, questioning whether forcing forward slashes in `xdg_config_home_for()` on Windows belongs in this topic and proposing a dedicated branch for the global scope in `config_with_options()`.

### worktree repair: detect relative path in .git file correctly
Yoichi NAKAYAMA's v4 fixes `git worktree repair` failing to convert relative cross-references to absolute paths when `worktree.useRelativePaths=false`, because `read_gitfile_gently()` always resolves to an absolute path. The fix introduces `read_gitfile_raw()` and has `repair_gitfile()` validate with `is_git_directory()`. Junio C Hamano reviewed it as "looks very good to me" and said he will mark it for `next` unless further comments arrive.

### Negative pathspecs in `git ls-files` and `git add`
Diogo Castro's patch fixes a bug in `dir.c` where the common prefix computed from only positive pathspecs is stripped from all pathspecs, so a negative pathspec shorter than that prefix degenerates to an empty string that matches every file. Junio C Hamano questioned whether the common prefix should be computed across both positive and negative pathspecs, while Diogo objected to extending the stripping to negative pathspecs, explaining the optimization avoids unnecessary working-directory walks.

### Use-after-free in `git stash show --src-prefix`
Nicolas Le Cam reported that `git stash show --src-prefix=a/ --dst-prefix=b/` prints fragments of freed heap memory instead of the requested prefixes in the diff header, reproducible on 2.52.0 through 2.54.0. He traced the regression to commit 3ea35c64b, which set `.free_removed_argv_elements = 1` in `show_stash()`; because `OPT_STRING_F` stores pointers into argv elements rather than copying them, freeing those elements leaves `diff_options.a_prefix`/`b_prefix` dangling. He warned the same hazard applies to any diff option parsed with `OPT_STRING*` into a `struct diff_options` field.

### Worktree lifecycle hooks
Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove`, and `post-worktree-move` hooks in `builtin/worktree.c`, motivated by tools that need reliable triggers when linked worktrees appear, move, or are removed. He argued that a wrapper around Git is insufficient because the component needing notification does not control the callers, and proposed consolidating the three hooks into a single `post-worktree` hook with a fixed interface issuing one `remove` event per entry actually pruned.

### sequencer honoring --empty when a fixup empties its target
Farid Zakaria posted v4 of the patch making `git rebase --autosquash --empty=drop` drop a commit that a `fixup!` or `squash!` empties. V4 restricts dropping to commits that become empty, allows only the final fixup of a chain to drop the emptied commit, and extends `commit_staged_changes()` to honor `--empty=drop` for a conflicted fixup resolved by undoing its target.

## In brief
- **reftable stack reload avoidance**: Karthik Nayak posted v2 avoiding stack reloads inside a locked reference transaction, adding a `locked` bit to `reftable_addition` with benchmarks showing 15-20% improvement; Jeff King reviewed patch 4/4 calling the corrected benchmarks "much nicer", and Nayak defended keeping the lock-state bitfield in `struct reftable_addition` rather than moving it to `reftable_stack`.
- **sequencer ODB-release**: Phillip Wood called the patch a fine immediate fix but suggested `-c gc.auto=false` when forking `git commit`, and both he and Junio C Hamano endorsed studying automatic ODB closure when forking Git commands.
- **push.negotiate=shallow**: Patrick Steinhardt proposed a `push.negotiate=shallow` option, enabled by default for shallow clones, instead of new send-pack logic for withholding the shallow boundary tree.
- **die_for_incompatible_opts()**: Junio C Hamano proposed a variadic API for unbounded option checks, but after Jeff King called the varargs interface less safe and René Scharfe proposed alternatives, Junio discarded the topic for now.
- **format-patch --[no-]range-diff-notes**: Kristoffer Haugsbakk proposed the option but it was dropped after Junio C Hamano questioned whether `--no-notes --range-diff-notes` could be defined simply.
- **symbolic-ref exit codes**: Reviewers disagreed over pinning `git symbolic-ref` exit codes 1 vs 128; Nikolaus Schuetz favored not testing the exact code and offered to sync the adoc NOTES.
- **configurable stash ref**: Phillip Wood answered the RFC, preferring `git stash pop` options for the latest stash on the current branch or worktree, plus optional `git stash list` filtering; he also proposed a per-branch stash pop option as an alternative, which Junio C Hamano cautioned against relying heavily on stash messages.
- **git reset --pull --hard**: Artur Bieniek proposed the inverse design; Junio C Hamano suggested an alias may be the right feature for this use case.
- **branch -d protecting local upstreams**: Elijah Newren reported a hole in Harald Nordgren's series where `git branch -d A B` protected B but deleted A, leaving surviving B with a missing upstream; Junio had earlier objected that changing `-d` semantics would break workflows.
- **odb alternates at creation time**: Patrick Steinhardt posted an 8-patch series writing alternates at object database creation time via `odb_source_create_on_disk()`, fixing a worktree-alternates bug; Toon Claes reviewed it and Junio asked whether the append-mode write should use the lockfile API.
- **git am --3way postimage blobs**: Nikita Leshenko proposed recording postimage blobs of cleanly applied patches; Junio C Hamano rejected the eager overhead in favor of lazy reconstruction.
- **checkout -m autostash refinement**: Phillip Wood reviewed Harald Nordgren's series, welcoming the conflict-result blank line but pushing back on gating the autostash retry on tracked local changes.
- **git add -e**: Junio C Hamano marked for `next` the patch replacing the `run_command()` subprocess in `builtin/add.c` with a direct `apply_all_patches()` call.
- **zsh completion**: Junio C Hamano marked for `next` the patch supporting `git -C <path>` by skipping global options when locating the command.
- **leaked git-header state**: Junio C Hamano declined the patch against leaked abandoned git-header state in `apply`, saying the cross checks are sanity checks.
- **worktree repair is_git_directory**: Yoichi Nakayama explained moving the `is_git_directory()` check out of `read_gitfile_raw()` into worktree.c; Junio confirmed the functional split is fine.
- **you_still_use_that() v3**: Junio C Hamano posted v3 fixing "mailing list" and correcting two tests to look for "will be removed soon".
- **git-svn v2**: Wesley Schwengle submitted v2 silencing v1-layout migration noise and adding a `NO_GIT_SVN` Makefile knob.
- **CI PR grouping**: Harald Nordgren posted v2 grouping pull request workflow runs by PR number with `cancel-in-progress`; Junio flagged that `push` and `pull_request` events now fall into separate concurrency groups.
- **http.sslVerifyStatus tests**: Grayson Gordon asked Junio to clarify a preference between Patrick's thorough tests and succinct ones; Junio asked which parts make Patrick uncertain.
- **pushcert documentation**: Lorenz Leutgeb submitted a one-line patch adding `'if-asked'` as a valid value for the `pushcert` option in `Documentation/gitremote-helpers.adoc`.
- **typo fixes**: Hardik Kumar posted an NFC patch fixing spelling in `versioncmp.c`, `git-gui/git-gui.sh`, and `t/t0022-crlf-rename.sh`.
- **AI DCO discussion**: Weijie Yuan retracted earlier remarks and reframed the question around AI-assisted DCO risk; Oswald Buddenhagen argued due diligence cannot be assumed with AI tools.

## Looking ahead
The `git history squash` series, now at 90 replies, will likely continue drawing review attention on hexadecimal case handling and range validation. brian m. carlson's Git 3.0 lowercase-hex RFC is queued and will see further discussion on security implications and remaining `HEX_KIND_MIXED` uses. K Jayatheerth's `git repo info` path-keys series will be requeued on newer master to fix the build break, and Elijah Newren's packfile recovery series awaits Junio C Hamano's decision on marking it for `next`. Domen Kožar's worktree lifecycle hooks proposal and Nicolas Le Cam's stash show use-after-free report were both posted late in the week and will likely see initial review next period.