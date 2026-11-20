## Long threads

### `git history squash` subcommand for folding a commit range

*Long thread: 90 replies so far from 9 participants over 73 days.*

Harald Nordgren's series adding `git history squash <revision-range>` dominated the month, first ejected from `next` after Phillip Wood exposed critical reachability flaws, then continuing through v14 with active refinement. The debate centered on hexadecimal case handling in `fixup!` subject matching: Phillip Wood argued `starts_with()` suffices for `git commit --fixup` output, while Junio C Hamano insisted the match must remain case-sensitive, aligning with brian m. carlson's broader lowercase-only direction for Git 3.0. By month's end the series remained in active review with trailer ordering and test usage being polished.

### Converting build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread: 44 replies so far from 6 participants over 28 days.*

D. Ben Knoble's series converting the compile-time `USE_NSEC` macro into the runtime `core.useNanosec` configuration reached v6, with Patrick Steinhardt declaring it ready for `next`. Jeff King clarified that `NO_NSEC` builds fake nsec entries as zero, making the setting a noop there. Junio C Hamano raised a concern about `repo_config_values()` potentially hitting a `BUG()` on repositories other than `the_repository` during submodule recursion, and Knoble argued recent work should lift that restriction rather than paper over it. By week's end Knoble declined a final reroll absent strong demand, leaving the series in a holding pattern.

### `git-replay(1)` gains a `--linearize` option

*Long thread: 39 replies so far from 7 participants over 86 days.*

Toon Claes's series adding `--linearize` to `git-replay(1)` resolved its multi-branch ambiguity by restricting the option to a single positive ref, eliminating the design impasse that had blocked it in `next`. The week brought v9 with rephrased error messages, and Elijah Newren reviewed the series, stating "I think this version is good to go" and adding a `Reviewed-by` trailer, effectively clearing the series for integration.

### Moving libgit.a sources into a `lib/` directory

*Long thread: 38 replies so far from 12 participants over 124 days.*

Patrick Steinhardt's RFC to move roughly 700 libgit.a source files into a `lib/` directory drew sustained debate. Junio C Hamano countered that `lib/` is an arbitrary intermediate layer that may worsen navigation, while Michael Montalbo proposed a script-based grouping of 231 files into 14 directories using commit-history labels. The debate remained unresolved by month's end.

### `git repack --drop-filtered` for partial clones

*Long thread: 37 replies so far from 5 participants over 51 days.*

Siddharth Shrimali's GSoC series adding `--drop-filtered` and `--dry-run` to `git repack` advanced through v5, addressing a type-mismatch bug in `list_objects_filter__filter_oidset()`. Christian Couder approved v5, and Junio C Hamano asked whether to mark the topic for `next`. Samuel Bronson reported successful testing on a real llvm-project clone, proposing an RFC change to skip index-referenced blobs with a warning rather than aborting, which Shrimali agreed to handle via a separate skip-set.

### `receive-report` hook for `git-receive-pack`

*Long thread: 37 replies so far from 5 participants over 18 days.*

Karthik Nayak's series introducing the `receive-report` hook advanced through v7, adopting an `enum report_status_version` to replace a `bool detailed_report` parameter after Junio C Hamano's suggestion. The hook runs after the reference transaction but before the client report, reading the pkt-line report on stdin; a non-zero exit marks all references rejected. Patrick Steinhardt offered nits on switch statements that were folded into v7.

### Packfile recovery when a MIDX names a removed pack

*Long thread: 36 replies so far from 5 participants over 15 days.*

Elijah Newren's series fixing objects reported missing when a stale multi-pack-index names a pack removed by concurrent geometric repacking converged after Jeff King objected that `OBJECT_INFO_QUICK` readers accept false negatives by design. v3 dropped the QUICK-recovery patch, instead fixing `git mktree --batch` to stop passing QUICK and renaming `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return. Derrick Stolee reviewed patch 4/4 with "Good patch. LGTM.", and Jeff King agreed with Junio C Hamano's proposal to mark the series for `next`.

### Pluggable packfile writes for `receive-pack`

*Long thread: 35 replies so far from 3 participants over 16 days.*

Justin Tobler's series adding a generic `odb_transaction_write_pack()` interface advanced to v5, fixing a critical lockfile lifecycle flaw where `.keep` files were moved during commit but tempfile structures still pointed to pre-migration paths. Patrick Steinhardt approved using transaction flags such as `ODB_TRANSACTION_RECEIVE`, while Junio C Hamano flagged reading `receive.unpackLimit` in generic object-layer code as a layering violation and suggested `odb-file.unpackLimit`.

### Ambiguous remote branch message in `worktree add`

*Long thread: 32 replies so far from 6 participants over 20 days.*

Yoichi NAKAYAMA's series improving the error when `git worktree add` matches multiple remote-tracking branches reached v10, adding a fourth commit making `--guess-remote` fail on multiple matches instead of falling back to HEAD. Junio C Hamano reviewed v10 favorably and stated he will mark it for `next`.

### New path keys for `git repo info`

*Long thread: 31 replies so far from 5 participants over 52 days.*

K Jayatheerth's GSoC series adding `path.*` keys to `git repo info` reached v5 with the `path.cdup` key, but SZEDER Gábor flagged a build break because the series depends on `repo->prefix`. Junio C Hamano identified that `get_superproject_working_tree()` relies on the process working directory rather than the target repository, causing empty output with `--git-dir`; Jayatheerth committed to updating it to take a `repo` parameter in v6.

### Pluggable packfile generation via the object database

*Long thread: 31 replies so far from 6 participants over 15 days.*

Patrick Steinhardt's v4 made packfile generation pluggable for `git-upload-pack`, `git-send-pack`, and `git-bundle` via a new `odb_generate_pack()` interface, fixing a use-after-scope bug Elijah Newren identified. Karthik Nayak gave a clean review, though Elijah proposed an alternative for the `the_repository` cleanup in `bundle.c` which Patrick defended.

### Eagerly loading ODB alternates

*Long thread: 31 replies so far from 5 participants over 12 days.*

Patrick Steinhardt's v3 initializes alternates up front when the object database is created. Karthik Nayak affirmed the rationale that `core.sharedRepository` must be known when the ODB is created, while Junio C Hamano questioned `eq()` semantics for filesystems with differing case sensitivity, leading to agreement that the existing hashing can stay as a best-effort mechanism.

### Repartitioning checkout/switch/restore away from monolithic `checkout_main()`

*Long thread: 27 replies so far from 5 participants over 8 days.*

Junio C Hamano's eight-patch draft reworks `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` to dispatch directly to `checkout_branch()` or `checkout_paths()`. Karthik Nayak agreed that patch 6 makes patch 1's removal of `cb_option` coherent, and Hamano confirmed the series "merely scratches the surface" since a whole command like `git restore` may still be too coarse as reusable code.

### `checkout -m`: refine autostash fallback

*Long thread: 26 replies so far from 4 participants over 42 days.*

Harald Nordgren's series making autostash conflicts in `sequencer.c` exit with status 1 reached v5, queued after Hamano said both patches look good. The remaining discussion centered on CI style tooling, with Nordgren noting the clang-format job misses mismatched-brace issues.

### Refuse `git commit --amend` and partial commits during conflict resolution

*Long thread: 26 replies so far from 3 participants over 8 days.*

Elijah Newren's series extending the refusal of `git commit --amend` and partial commits beyond merge and cherry-pick to `am`, `revert`, and rebase conflict stops reached v4. Phillip Wood reviewed the `FROM_REBASE_NOW_EMPTY` rename as clearer but expressed mixed feelings about permitting a partial commit when a rebase pick becomes empty.

### trace2 stops allowing die()

*Long thread: 25 replies so far from 6 participants over 49 days.*

Derrick Stolee's series bans die()-capable helpers from the low-level trace2 API via a root-level `banned-die.h` header. Jeff King argued that chasing transitive die() calls would need static analysis or a ground-up rewrite, though he is not opposed to the header in the meantime.

### Git 3.0: restrict hex object IDs to lowercase only

*Long thread: 25 replies so far from 6 participants over 28 days.*

brian m. carlson's RFC proposing to remove uppercase hex from object IDs in Git 3.0 drew maintainer scrutiny. When Phillip Wood asked for the security problems behind rejecting uppercase, brian cited policy checks against the lowercase form being bypassed by uppercase input. Junio C Hamano and Elijah Newren also flagged remaining `HEX_KIND_MIXED` uses in individual patches.

### diff hunk provider interface rethought

*Long thread: 25 replies so far from 4 participants over 94 days.*

Michael Montalbo retracted the "diff provider" abstraction introduced in RFC v7, saying it shoehorns several concepts together, and decided to return to the v6 line for future re-rolls while ejecting the diff-hunks store patches.

### Make t/lib-httpd CGI test helpers concurrency-safe

*Long thread: 23 replies so far from 3 participants over 58 days.*

Michael Montalbo's series fixing race conditions in `apply-one-time-script.sh` and `http-429.sh` reached v5, which Patrick Steinhardt reviewed as looking good.

### Fixing the O(N^2) packfile-loading regression

*Long thread: 22 replies so far from 5 participants over 6 days.*

Johannes Schindelin's patch fixing `packfile_store_add_pack()` linearly scanning the pack list on every insertion was marked for `next` by Junio C Hamano. Jeff King identified that `packfile_list_prepend()` has the same O(N^2) problem and concluded MRU updates should use constant-time movement within a doubly-linked list.

### `you_still_use_that()` deprecation instructions in usage.c

*Long thread: 8 replies so far from 2 participants over 1 day.*

Junio C Hamano reworded the deprecation text to "'%s' will be removed soon" and "If you need a replacement:". Michael Montalbo agreed the change makes sense.

### builtin command `repo` parameter

*Long thread: 7 replies so far from 3 participants over 1 day.*

Hardik Kumar posted a patch replacing global `the_repository` uses with the previously-UNUSED `repo` parameter. Junio C Hamano countered that mixing `the_repository` and `repo` in one function is less consistent, and instead proposed replacing the `struct repository *` parameter of all builtins with `bool has_repo`.

### doc: cherry-pick: note `--no-commit` skips `CHERRY_PICK_HEAD`

*Long thread: 11 replies so far from 4 participants over 2 days.*

Aleksei Sviridkin's documentation-only change notes that `cherry-pick --no-commit` skips `CHERRY_PICK_HEAD`. v2 dropped the t3507 test after Patrick Steinhardt argued tests need stronger justification, while Phillip Wood disputed that existing tests cover the conflicting case.

## The period in brief

August 2026 was a heavy, eventful month on the Git mailing list, with sustained traffic across all six weeks and major milestones in the object database abstraction, branch management, and security hardening. Readers should not miss the landing of `git branch --delete-merged` in `next` after 25 iterations, the ejection and continued refinement of `git history squash`, and the graduation of two ODB abstraction series to `master` making `git receive-pack` backend-agnostic.

## Key developments

### ODB abstraction reaches major milestones

Patrick Steinhardt and Justin Tobler drove the pluggable object database effort forward on multiple fronts. Two series graduated to `master`: Steinhardt's six-patch series making on-disk ODB structures pluggable, and Tobler's seven-patch series extending ODB transactions to support packfile writes in `git receive-pack`, making the first network-facing command fully backend-agnostic. Additional series advanced pluggable packfile generation for transport via `odb_generate_pack()` and eagerly loading ODB alternates at creation time, with review focusing on transaction flags, `.keep` file lifecycle, and config naming.

### `git branch --delete-merged` lands in `next`

Harald Nordgren's 25-patch series adding `git branch --delete-merged` graduated to `next` after six months of review. The feature provides order-independent stacked-branch protection, per-branch opt-out via `branch.<name>.deleteMerged=false`, and `--dry-run` preview, addressing local branch clutter while preserving safety checks for checked-out branches and push divergence.

### `git history squash` ejected then refined

Toon Claes's `git history squash` was ejected from `next` after Phillip Wood exposed critical reachability flaws, prompting Junio to propose a temporary moratorium on new `seen` topics without substantive review. Harald Nordgren's separate `git history squash` series continued through v14 with active refinement of hex-case handling and trailer ordering, remaining in review at month's end.

### Security and correctness fixes

Johannes Schindelin posted a fix for CVE-2026-32631 in Git for Windows preventing credential exfiltration via symlink auto-detection. Tim Wiederhake's `gitk` custom commands patch faced a critical shell-injection flaw via unquoted placeholder substitution. Elijah Newren's series fixed objects reported missing when a stale multi-pack-index names a removed pack, and Nicolas Le Cam reported a use-after-free in `git stash show --src-prefix` traced to commit 3ea35c64b.

### Performance optimizations land

Kristofer Karlsson and Tian Yuchen's `paint_down_to_common()` optimization providing 100-1000x speedup for asymmetric merge-base queries was queued in Junio's tree. Johannes Schindelin fixed an O(N^2) regression in packfile loading that slowed `git rev-parse` from 0.4s to 4.5s with many packs. Karthik Nayak's reftable stack reload avoidance showed 15-20% improvement for `update-ref --stdin` with 50,000 refs.

### Git 3.0 planning and lowercase-hex RFC

brian m. carlson's RFC proposing lowercase-only hex object IDs for Git 3.0 sparked philosophical debate about robustness versus security, with Junio C Hamano and Jeff King flagging edge cases in dumb HTTP transport and object payload hashing. A separate thread discussed naming the release after Git 2.56, with brian suggesting Git 3.0 might be appropriate given remaining SHA-256 forge support considerations.

### New hooks and transport features

Karthik Nayak's `receive-report` hook series reached v7, running after the reference transaction but before the client report. Elijah Newren's shallow-clone push optimization introduced `push.shallowExcludeBoundary` to avoid sending the entire toplevel tree. Domen Kožar proposed worktree lifecycle hooks (`post-worktree-add`, `post-worktree-remove`, `post-worktree-move`).

## In brief

**`git replay --linearize`** -- Toon Claes's v9 resolved the multi-branch ambiguity by restricting to a single positive ref, earning Elijah Newren's `Reviewed-by` and clearing for integration.

**`git add --resolved`** -- Junio's series staging only paths with conflict markers removed sparked debate with Michael Montalbo over whether the name conflates selection with policy.

**`git stash reword`** -- Emin Özata's feature remains blocked pending Junio's proposed `refs_reflog_edit_in_bulk()` API for atomic reflog replacement.

**`git cat-file --batch-command`** -- Pablo Sabater's GSoC series adding `%(objecttype)` support reached v6 and is ready for integration.

**`git bisect --reset-when-found`** -- Harald Nordgren's series adding auto-reset after finding the first bad commit is ready for `next`.

**`git rebase --update-refs` symref bugfix** -- Son Luong Ngoc's series resolved its design debate after Phillip Wood provided TopGit evidence and is ready for merging.

**`git worktree add` usability** -- Yoichi NAKAYAMA's series improving ambiguous remote branch error messages reached v10 and will be marked for `next`.

**`git worktree repair`** -- NAKAYAMA's v4 fixing relative path detection in `.git` files was reviewed as "looks very good to me".

**`git config --global` XDG** -- Delilah Ashley Wu's v2 makes `git config --global` read both `$HOME/.gitconfig` and the XDG config file.

**`git fast-import` libification** -- Christian Couder's 12-patch series refactoring to use parse-options API is nearly ready.

**`USE_NSEC` runtime config** -- D. Ben Knoble's series converting build-time macro to `core.useNanosec` reached v6 but stalled on submodule recursion concerns.

**Sparse-index use-after-free** -- Shlok Kulshreshtha's fix for `get_oid_with_context_1()` is ready for integration.

**`git send-email`** -- Harald Nordgren's bugfix for missing "Subject:" headers was approved for `next`.

**`git history` completion** -- Vincent Mailhol's v2 added Bash completion with D. Ben Knoble advocating simplification.

**`git maintenance` regression** -- David Lin's patch restoring `"none"` strategy behavior is queued for `next`.

**`git add -e` refactoring** -- Gatla Vishwahwar Reddy's patch replacing subprocess with internal API is approved for `next`.

**`git index-pack` race fix** -- Ted Nyman's fix for concurrent download corruption is ready for merge.

**Promisor-pack performance** -- Arijit Banerjee's 15-26% speedup patch stalled on AI-assisted contribution policy debate.

**`diff-provider` RFC** -- Michael Montalbo retracted v7 and returned to v6 line for narrower pkt-line interface.

**`git organize` RFC** -- Montalbo posted 14-patch RFC introducing `git organize` command with `.gitorganize` file.

**`git pull --hard` vs `--reset`** -- Discussion shifted toward Phillip Wood's `git pull --reset` alternative.

**`report` hook** -- Karthik Nayak's v2 of receive-report hook; Junio agreed failed hook should explicitly decline push.

**Trailer URL fix** -- Kristoffer Haugsbakk's v3 making trailer parsing fail on `://` separators marked for `next`.

**Credential libsecret fix** -- Daniel Martí defended explicit secret-loading fix against NULL-check fallback.

**`stash.ref` configuration** -- Vladimir Sitnikov proposed knob for isolated stash stacks per worktree.

**`submodule.active`** -- Tilak Raaz posted v4 warning on valueless config.

**`format-rev` options** -- Haugsbakk added `--abbrev`, `--color`, `--date` with Junio catching assertion-order bug.

**`bitmapPseudoMerge.*`** -- kapicharly documented experimental config options.

**`git squash`** -- Matej Dujava and Bradley Morgan supported proposed command.

**`git var` extensions** -- Andrew Pleeter's `git ident` proposal rejected in favor of extending `git var`.

**`early_scan_options()`** -- Christian Couder posted six-patch series adding sub-API to parse-options.

**`git history split` SIGSEGV** -- Jinbao Chen fixed crash when parent tree object missing.

**`git imap-send --draft`** -- Wolfgang Faust added `--draft` option for IMAP `\Draft` flag.

**Rust in Windows CI** -- Johannes Schindelin posted series enabling Rust in Windows CI jobs.

**Outreachy December 2026** -- Christian Couder announced Community Application with two intern project ideas.

**AI-generated patch refusal** -- brian m. carlsson declined to review Gemini-generated treewide migration over DCO concerns; Junio endorsed the stance.

## Looking ahead

Several series posted late in August will see review in September: Karthik Nayak's v7 of the `receive-report` hook rebased on master, Toon Claes's `git-replay --linearize` with Elijah Newren's `Reviewed-by`, and Elijah Newren's shallow-push series with the tri-state `push.shallowExcludeBoundary`. D. Ben Knoble's `core.useNanosec` series awaits resolution of the `repo_config_values()` recursion concern, while K Jayatheerth's `git repo info` path series needs the `get_superproject_working_tree()` fix in v6. The `git history squash` hex-case debate will likely continue influenced by brian m. carlson's Git 3.0 lowercase-only RFC, and the root-directory reorganization debate between Patrick Steinhardt and Junio C Hamano remains unresolved.