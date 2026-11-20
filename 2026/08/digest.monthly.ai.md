## Long threads

### `git history squash` subcommand for folding a commit range
*Long thread: 90 replies so far from 9 participants over 73 days.*
Harald Nordgren's series adding `git history squash <revision-range>` was ejected from `next` early in the month after Phillip Wood exposed critical reachability flaws, but development continued through v14. The debate centered on hexadecimal case handling in `fixup!` subject matching, with Junio C Hamano insisting on case-sensitive matching to align with brian m. carlson's lowercase-only direction for Git 3.0. By month's end the series remained in active review, with trailer ordering and `Signed-off-by` details being refined.

### Convert build-time `USE_NSEC` into runtime `core.useNanosec`
*Long thread: 44 replies so far from 6 participants over 28 days.*
D. Ben Knoble's series converting the compile-time `USE_NSEC` macro into the runtime `core.useNanosec` configuration reached v6, with Patrick Steinhardt declaring it ready for `next`. Junio C Hamano raised a concern about `repo_config_values()` potentially hitting a `BUG()` during submodule recursion, and Knoble argued recent work should lift the restriction rather than paper over it. By week's end Knoble declined a final reroll absent strong demand, leaving the series in a holding pattern.

### git-replay(1) gains a `--linearize` option
*Long thread: 39 replies so far from 7 participants over 86 days.*
Toon Claes's series adding `--linearize` to `git-replay(1)` resolved its multi-branch ambiguity by restricting the option to a single positive ref in v8, aligning with Elijah Newren's minimal-fix proposal. The week brought v9 with rephrased error messages, and Newren reviewed the series stating "I think this version is good to go," adding a `Reviewed-by` trailer and effectively clearing it for integration.

### Moving libgit.a sources into a `lib/` directory
*Long thread: 38 replies so far from 12 participants over 124 days.*
Patrick Steinhardt's RFC to move roughly 700 libgit.a source files into a `lib/` directory drew Junio C Hamano's counter that the directory is an arbitrary intermediate layer, while Michael Montalbo proposed a script-based grouping of 231 files into 14 directories using commit-history labels. The debate remained unresolved by month's end, with Steinhardt emphasizing discoverability for newcomers and Hamano warning the move may worsen navigation.

### repack: add `--drop-filtered` to reclaim space in partial clones
*Long thread: 37 replies so far from 5 participants over 51 days.*
Siddharth Shrimali's GSoC series adding `--drop-filtered` and `--dry-run` to `git repack` for partial clones advanced through v5, fixing a type-mismatch bug and receiving Christian Couder's approval. Samuel Bronson tested it on a real llvm-project clone, finding it greatly reduced object database size, and proposed skipping index-referenced blobs with a warning rather than aborting. The series is ready for `next` with remote verification and a drop log deferred as follow-up work.

### hook: introduce the `receive-report` hook
*Long thread: 37 replies so far from 5 participants over 18 days.*
Karthik Nayak's series adding a `receive-report` hook for `git-receive-pack` advanced through v7, adopting an `enum report_status_version` to replace a boolean parameter after Junio C Hamano's suggestion. The hook runs after the reference transaction but before the client report, with a non-zero exit marking all references rejected. Patrick Steinhardt offered nits on switch statements that were folded into v7.

### packfile recovery when a MIDX names a removed pack
*Long thread: 36 replies so far from 5 participants over 15 days.*
Elijah Newren's series fixing objects reported missing when a stale multi-pack-index names a pack removed by concurrent geometric repacking brought v3, which dropped the `OBJECT_INFO_QUICK` recovery patch after Jeff King's objection and instead fixed `git mktree --batch` to stop passing QUICK. Derrick Stolee reviewed patch 4/4 with "Good patch. LGTM," and Jeff King agreed with Junio C Hamano's proposal to mark the series for `next`.

### Pluggable packfile writes for `receive-pack`
*Long thread: 35 replies so far from 3 participants over 16 days.*
Justin Tobler's series adding a generic `odb_transaction_write_pack()` interface so `git-receive-pack` writes through the ODB transaction layer reached v5, fixing a critical lockfile lifecycle flaw where `.keep` files were moved during commit but tempfile structures still pointed to pre-migration paths. Patrick Steinhardt approved using transaction flags like `ODB_TRANSACTION_RECEIVE`, while Junio C Hamano flagged reading `receive.unpackLimit` in generic code as a layering violation.

### worktree add message for ambiguous remote branch names
*Long thread: 32 replies so far from 6 participants over 20 days.*
Yoichi NAKAYAMA's series improving the error when `git worktree add` matches multiple remote-tracking branches reached v10, adding a commit making `--guess-remote` fail on multiple matches instead of falling back to HEAD. Junio C Hamano reviewed v10 favorably and stated he will mark it for `next`, noting the multiple-match-but-success case remains a separate bug.

### repo: add path keys to `git repo info`
*Long thread: 31 replies so far from 5 participants over 52 days.*
K Jayatheerth's GSoC series adding `path.*` keys to `git repo info` reached v5 with the addition of `path.cdup`, but SZEDER Gábor flagged a build break because the series depends on `repo->prefix`. Junio C Hamano identified that `get_superproject_working_tree()` relies on the process working directory rather than the target repository, causing empty output with `--git-dir`; Jayatheerth committed to fixing this in v6.

### Pluggable packfile generation via the object database
*Long thread: 31 replies so far from 6 participants over 15 days.*
Patrick Steinhardt's v4 making packfile generation pluggable for `git-upload-pack`, `git-send-pack`, and `git-bundle` via `odb_generate_pack()` fixed a use-after-scope bug Elijah Newren identified in `odb_source_files_generate_pack()`. Karthik Nayak gave a clean review, though Newren proposed an alternative for `the_repository` cleanup in `bundle.c` which Steinhardt defended.

### Eagerly loading ODB alternates
*Long thread: 31 replies so far from 5 participants over 12 days.*
Patrick Steinhardt's v3 initializing alternates up front when the object database is created added a patch creating ref and object databases after repository config is written in `init_db()`. Karthik Nayak affirmed that `core.sharedRepository` must be known when the ODB is created, while Junio C Hamano questioned `eq()` semantics for filesystems with differing case sensitivity, leading to agreement that existing hashing can stay as best-effort.

### Repartition checkout/switch/restore away from monolithic `checkout_main()`
*Long thread: 27 replies so far from 5 participants over 8 days.*
Junio C Hamano's eight-patch draft reworking `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` to dispatch directly to `checkout_branch()` or `checkout_paths()` was reviewed by Karthik Nayak, who agreed patch 6 makes patch 1's removal of `cb_option` coherent. Hamano confirmed the series "merely scratches the surface" since a whole command like `git restore` may still be too coarse as reusable code.

### Refuse `git commit --amend` and partial commits during conflict resolution
*Long thread: 26 replies so far from 3 participants over 8 days.*
Elijah Newren's series extending the refusal of `git commit --amend` and partial commits beyond merge and cherry-pick to `am`, `revert`, and rebase conflict stops reached v4 with the `FROM_REBASE_NOW_EMPTY` rename. Phillip Wood approved the logic but expressed mixed feelings about permitting a partial commit when a rebase pick becomes empty, since `CHERRY_PICK_HEAD` and `.git/MERGE_MSG` are written at such stops.

### checkout -m: refine autostash fallback
*Long thread: 26 replies so far from 4 participants over 42 days.*
Harald Nordgren's series making autostash conflicts in `sequencer.c` exit with status 1 rather than 128 reached v5, queued after Hamano said both patches look good. The remaining discussion centered on CI style tooling, with Nordgren noting the clang-format job misses mismatched-brace issues.

### Git 3.0: restrict hex object IDs to lowercase only
*Long thread: 25 replies so far from 6 participants over 28 days.*
brian m. carlson's RFC proposing to remove uppercase hex from object IDs in Git 3.0 drew scrutiny when Phillip Wood asked for the security problems behind rejecting uppercase; brian cited policy checks against lowercase being bypassed and regexps failing to match uppercase IDs. Junio C Hamano and Elijah Newren flagged remaining `HEX_KIND_MIXED` uses in individual patches.

### diff hunk provider interface rethought
*Long thread: 25 replies so far from 4 participants over 94 days.*
Michael Montalbo retracted the "diff provider" abstraction from RFC v7, saying it shoehorns several concepts together, and decided to return to the v6 line for future re-rolls. He clarified goals for a narrower pkt-line interface to xdiff for user-configured alternatives and OID-only operation for callers needing only hunk header info.

### trace2 stops allowing die()
*Long thread: 25 replies so far from 6 participants over 49 days.*
Derrick Stolee's series banning die()-capable helpers from the low-level trace2 API via a root-level `banned-die.h` header drew Jeff King's comment that chasing transitive die() calls would need a ground-up rewrite, though he is not opposed to the header in the meantime.

### Make t/lib-httpd CGI test helpers concurrency-safe
*Long thread: 23 replies so far from 3 participants over 58 days.*
Michael Montalbo's series fixing race conditions in `apply-one-time-script.sh` and `http-429.sh` seen in flaky t5616 failures reached v5, which Patrick Steinhardt reviewed as looking good. Earlier criticism of verbose AI-generated commit messages led Montalbo to commit to writing prose by hand.

### Fixing the O(N^2) packfile-loading regression
*Long thread: 22 replies so far from 5 participants over 6 days.*
Johannes Schindelin's patch fixing an O(N^2) regression in `packfile_store_add_pack()` that slowed `git rev-parse` from 0.4s to 4.5s with many packs was marked for `next` by Junio C Hamano. Jeff King countered that `packfile_list_prepend()` has the same problem and concluded MRU updates should use constant-time movement within a doubly-linked list.

### doc: cherry-pick: note `--no-commit` skips `CHERRY_PICK_HEAD`
*Long thread: 11 replies so far from 4 participants over 2 days.*
Aleksei Sviridkin's documentation-only change noting that `cherry-pick --no-commit` skips `CHERRY_PICK_HEAD` saw v2 drop the t3507 test after Patrick Steinhardt argued tests need stronger justification, while Phillip Wood disputed that existing tests cover the conflicting case.

### `you_still_use_that()` deprecation instructions in usage.c
*Long thread: 8 replies so far from 2 participants over 1 day.*
Junio C Hamano reworded the deprecation text to "'%s' will be removed soon" and "If you need a replacement:"; Michael Montalbo agreed the change makes sense and suggested stressing that the command is not returning.

### builtin command `repo` parameter
*Long thread: 7 replies so far from 3 participants over 1 day.*
Hardik Kumar posted a patch replacing global `the_repository` uses with the previously-UNUSED `repo` parameter in several `cmd_foo()` call sites. Junio C Hamano countered that mixing `the_repository` and `repo` in one function is less consistent, and instead proposed replacing the `struct repository *` parameter of all builtins with `bool has_repo`.

## The period in brief

August 2026 was a month of heavy, technically dense traffic on the Git mailing list, with multiple long-running series reaching decision points and several architectural efforts hitting major milestones. The ODB abstraction effort graduated key series to `master`, `git branch --delete-merged` finally landed in `next` after 25 iterations, and security fixes for Git for Windows and `gitk` were queued. Readers should not miss the pluggable packfile interfaces for `receive-pack` and transport, the ongoing `git history squash` saga, and the procedural debates over AI-assisted contributions.

## Key developments

### ODB abstraction reaches major milestones
Patrick Steinhardt and Justin Tobler advanced the pluggable object database effort with two series landing in `master`: Steinhardt's six-patch series making on-disk ODB structures pluggable, and Tobler's seven-patch series extending ODB transactions to support packfile writes in `git receive-pack`, making it the first network-facing command fully backend-agnostic. Follow-up series advanced pluggable packfile generation via `odb_generate_pack()` and eager loading of alternates at ODB creation time, while Steinhardt proposed moving alternates handling into the "files" backend itself.

### `git branch --delete-merged` lands in `next`
Harald Nordgren's 25-patch series adding `git branch --delete-merged` graduated to `next` after six months of review, providing safe automated local branch cleanup with order-independent stacked-branch protection, per-branch opt-out via `branch.<name>.deleteMerged=false`, and `--dry-run` preview. The feature addresses long-standing branch clutter while preserving safety checks for checked-out branches, missing upstreams, and push divergence.

### Security fixes and vulnerability disclosures
Johannes Schindelin posted a fix for CVE-2026-32631 in Git for Windows preventing credential exfiltration via symlink auto-detection triggering SMB connections to UNC paths, released in v2.53.0(3) and queued for upstream. Tim Wiederhake's `gitk` custom commands patch was found to interpolate placeholders into shell commands without quoting, enabling arbitrary command execution via malicious commit titles. Johannes Schindelin also announced Git for Windows 2.55.0(4) fixing CVE-2026-62960 in bundle URI handling, while Xavier Morel reported `git push --delete --porcelain` outputting human-readable instead of machine-readable errors.

### Performance optimizations across subsystems
Kristofer Karlsson and Tian Yuchen's `paint_down_to_common()` optimization providing a 100-1000x speedup for asymmetric merge-base queries was queued in Junio's tree. Karthik Nayak's reftable stack reload avoidance when locked showed a 15-25% improvement for `update-ref --stdin` with 50,000 refs. Toon Claes's Bloom filter reuse for `git last-modified` offered a 3.7x speedup, though the series was later ejected from `seen` after CI failures.

### AI-assisted contributions and review culture debates
The project grappled with how to handle AI-generated code after brian m. carlsson declined to review a treewide `utime.h` migration generated by Gemini Flash, arguing it cannot satisfy the Developer's Certificate of Origin requirement; Junio C Hamano endorsed the stance as prudent. Separately, Tim Wiederhake's `gitk` patch faced scrutiny after Johannes Sixt flagged its AI-assisted origins, and Harald Nordgren and Phillip Wood clashed over the pace of review iterations in the `git history squash` thread.

### GSoC projects reach completion
Pablo Sabater's series extending `git cat-file --batch-command` to support `%(objecttype)` in remote-object-info queries reached v6 and is ready for integration, completing the remote-object-info protocol's core functionality. Siddharth Shrimali's `git repack --drop-filtered` series and K Jayatheerth's `git repo info` path keys series both advanced significantly, with the former receiving approval for `next` and the latter awaiting a fix for `get_superproject_working_tree()`.

## In brief

**`git stash reword` blocked** -- Emin Özata's feature remains blocked pending Junio C Hamano's proposed `refs_reflog_edit_in_bulk()` API for atomic reflog entry replacement.

**`git add --resolved`** -- Junio's series staging only paths with removed conflict markers is ready for integration after addressing feedback, despite Michael Montalbo's philosophical objections to the option name.

**`git rebase --update-refs` symref bugfix** -- Son Luong Ngoc's series resolved its design debate after Phillip Wood provided evidence that TopGit relies on HEAD symrefs, and is ready for merging.

**`git bisect --reset-when-found`** -- Harald Nordgren's series adding auto-reset after finding the first bad commit is ready for `next`.

**`git fast-import` libification** -- Christian Couder's 12-patch series refactoring to use the parse-options API is nearly ready with three minor issues remaining.

**`USE_NSEC` runtime config** -- D. Ben Knoble's conversion to `core.useNanosec` is stalled on a `repo_config_values()` recursion concern for submodules.

**Sparse-index use-after-free** -- Shlok Kulshreshtha's fix for `get_oid_with_context_1()` is ready for integration.

**`git worktree add` data-loss bug** -- Matthias Aßhauer fixed an out-of-bounds read in `worktree_basename()` that could cause recursive deletion of the current working directory.

**`git worktree repair` relative paths** -- Yoichi NAKAYAMA's v4 fixing relative `.git` cross-references was marked for `next` by Junio C Hamano.

**`git config --global` XDG** -- Delilah Ashley Wu's v2 making `git config --global` read both `$HOME/.gitconfig` and the XDG config file fixes a documentation discrepancy.

**`git send-email`** -- Harald Nordgren's bugfix for missing "Subject:" headers was approved for `next`.

**`git maintenance` regression** -- David Lin's patch restoring the documented `"none"` strategy behavior is queued for `next`.

**`git index-pack` race-condition fix** -- Ted Nyman's fix for concurrent download corruption in HTTP transport is ready for merge.

**`git mv` symlink-handling fix** -- Lucas Zamboni Orioli's series fixing misleading errors is ready for integration.

**`git add -e` refactoring** -- Gatla Vishwahwar Reddy's patch replacing a subprocess call with internal API is approved for `next`.

**Coccinelle performance** -- SZEDER Gábor's benchmarks show Coccinelle 1.3.1 is 4.5x slower than 1.1.1, challenging the Ubuntu image bump in CI.

**`git rebase` vs `git merge` `-X` inconsistency** -- zrzut01@poczta.fm reported that `git rebase -X <invalid-option>` silently ignores the option while `git merge` errors out.

**`git history` completion** -- Vincent Mailhol's v2 added Bash completion with D. Ben Knoble advocating simplification.

**`git-interpret-trailers` documentation** -- Kristoffer Haugsbakk's 11-patch series overhauling the man page is fully merged to `master`.

**`git var` extensions** -- Andrew Pleeter's `git ident` proposal was rejected in favor of extending `git var` with broken-down identity components.

**`git squash` proposal** -- Matej Dujava and Bradley Morgan supported the proposed command.

**Translation and documentation updates** -- Jiang Xin and Jean-Noël Avila continued l10n and synopsis-style conversion work; Kristoffer Haugsbakk posted multiple doc improvements including `gitdatamodel(7)` advertising and trailer URL parsing fixes.

**Test modernization** -- Junio C Hamano objected to mechanically converting `test -f` to `test_path_is_file` inside `if` conditionals.

**CI and build system** -- Jeff King proposed bumping the debian-11 CI job to debian-12; Johannes Schindelin posted patches enabling Rust in Windows CI jobs.

## Looking ahead

Several series posted late in August will see review in September: Toon Claes's `git-replay --linearize` v9 carrying Elijah Newren's `Reviewed-by`, Yoichi NAKAYAMA's worktree ambiguous-remote v10, and Elijah Newren's packfile recovery series all await `next` integration. The `git history squash` series will continue iterating on hex-case handling, while D. Ben Knoble's `core.useNanosec` series remains in limbo pending guidance on the `repo_config_values()` recursion concern. brian m. carlsson's Git 3.0 lowercase-only RFC and the naming discussion for the post-2.56 release will likely require PLC input to resolve, and Patrick Steinhardt's root-directory reorganization debate will continue as he and Junio C Hamano seek consensus on the `lib/` proposal.