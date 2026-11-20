## Long threads

### Moving libgit.a sources into a "lib/" directory

*Long thread: 37 replies so far from 12 participants over 121 days.*

Patrick Steinhardt's RFC proposes moving all libgit.a sources from the repository root into a `lib/` directory to shrink the top level from over 550 files to about 80, improving discoverability for newcomers. The debate has drawn contrasting views: SZEDER Gábor argued the move trades one annoyance for another, while Junio C Hamano countered that `lib/` is an arbitrary intermediate layer that fails to deliver meaningful organization and warned it may worsen navigation by forcing contributors to re-learn the structure later. Johannes Schindelin and brian m. carlson defended the bulk move, and Michael Montalbo proposed a script-based grouping of the 231 source files into 14 directories using commit-history labels and cscope call-graph analysis. The thread remains unresolved, with Steinhardt maintaining the newcomer-focused rationale despite Junio's concerns about rename fallout.

### `git repack --drop-filtered` for partial clones

*Long thread: 35 replies so far from 4 participants over 30 days.*

Siddharth Shrimali's GSoC series adds `--drop-filtered` and `--dry-run` to `git repack`, allowing partial clones to reclaim disk space by dropping locally cached promisor blobs above a size threshold while relying on lazy fetch to restore them. After multiple iterations addressing review feedback, v5 fixed a type-mismatch bug in `list_objects_filter__filter_oidset()` and clarified that remote verification, recency-based culling, and a drop log are follow-up work rather than prerequisites. By the end of the period, Christian Couder reviewed v5 positively, Junio C Hamano asked whether to mark the topic for `next`, and Shrimali confirmed the series is ready, with Elijah Newren and Kristofer Karlsson also declaring the related promisor object filtering series ready for advancement.

### Pluggable packfile writes for receive-pack

*Long thread: 28 replies so far from 3 participants over 9 days.*

Justin Tobler's series introduces a generic `odb_transaction_write_pack()` interface so `git receive-pack` writes incoming packfiles through the ODB transaction layer, making the existing `git-index-pack`/`git-unpack-objects` logic the "files" backend. The discussion has centered on `.keep` file cleanup after transactions reorder the ODB source list and on fixing a critical lockfile lifecycle flaw where tempfile structures pointed to pre-migration paths. Patrick Steinhardt noted the `write_pack()` callback needs to know which scenario it runs in, and Tobler confirmed that flags like `ODB_TRANSACTION_RECEIVE` already exist for this, with `ODB_TRANSACTION_FETCH` planned for `git fetch-pack` in a follow-up.

### `git repo info` path keys from the GSoC series

*Long thread: 21 replies so far from 4 participants over 32 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix` -- each supporting absolute and relative format suffixes. Earlier revisions renamed `superproject-working-tree` to `superproject-root`, dropped `path.objects` until pluggable ODB backends arrive, and moved the `prefix` member into `struct repository`. Lucas Seiki Oshiro questioned whether `path.git-prefix` belongs in `git repo info`, arguing it represents current directory information rather than repository information, and also asked why the `path.toplevel` test does not use the `test_repo_info_path` helper.

### Fixing the O(N^2) packfile-loading regression

*Long thread: 20 replies so far from 5 participants over 3 days.*

Johannes Schindelin's patch fixes a regression where `packfile_store_add_pack()` linearly scanned the pack list on every insertion, making loading N packs O(N^2) and slowing `git rev-parse --short HEAD` from 0.4s to 4.5s with 37,815 packs. The fix adds a `skip_dup_check` parameter to `packfile_list_append()`. Patrick Steinhardt agreed a hashmap already exists via `packfile_store_add_pack()` and approved the v2 patch, Junio C Hamano marked it for `next`, and Jeff King noted the map avoids all of `add_packed_git()` -- allocation and stat calls -- not just list insertion.

## The period in brief

The week of 2026/08/10 through 2026/08/16 saw heavy traffic across seven active days, with 81 emails on Monday alone and sustained discussion across regressions, security fixes, and long-running architectural efforts. The most urgent items were two regressions -- one in `git rebase` with submodules and commit graphs, another in `git maintenance` geometric repacking -- plus a critical security flaw in `gitk` custom commands. Readers should also note the completion of the `paint_down_to_common()` optimization series, the advancement of Justin Tobler's ODB transaction work for `git receive-pack`, and the heated debate over root-directory reorganization.

## Key developments

### Regressions in `git rebase` and `git maintenance` demand urgent attention

Two high-priority regressions surfaced during the week. A `git rebase` regression bisected to `bb5da75d61` causes fatal errors when submodules are involved because `lookup_commit_reference_gently()` resolves submodule OIDs against the main repository's commit graph; Patrick Steinhardt asked for confirmation whether the issue persists on `master` and a minimal reproducer, with `core.commitGraph=false` as a workaround. Separately, Stefan Haller reported two critical issues in `git maintenance` geometric repacking: concurrency failures where `prune_packed_objects()` removes fanout directories while concurrent writers create temporary files, and an auto-condition bug triggering 256x more aggressively than documented. Patrick Steinhardt proposed raising the default `geometric-repack.auto` threshold from 100 to 6700, which Justin Tobler endorsed as "trivially correct," though the underlying race condition remains unaddressed.

### Security flaw in `gitk` custom commands blocks feature

Tim Wiederhake's patch adding user-configurable custom commands to `gitk` contains a critical security flaw allowing arbitrary command injection via unquoted placeholder substitution. Junio C Hamano provided a concrete exploit example using a commit title containing shell metacharacters. The author is evaluating two fixes: reverting to direct command execution or implementing proper escaping for all interpolated values. This is a blocker for the feature, which aims to extend `gitk`'s workflow flexibility.

### ODB abstraction advances with transaction support for `git receive-pack`

Justin Tobler's nine-patch series extending ODB transactions to `git receive-pack` reached v3, addressing all feedback from v2. Key improvements include fixing a critical lockfile lifecycle flaw in the "files" backend via explicit `struct odb_source *` tracking, introducing `odb_transaction_finalize()` and `odb_transaction_commit_and_finalize_or_die()`, removing reliance on global variables, and adding `odb_transaction_write_pack()` as the generic interface completing the ODB abstraction for `git receive-pack`. The series is cooking in `seen`, with Patrick Steinhardt noting the `write_pack()` callback needs scenario flags, which Tobler confirmed exist as `ODB_TRANSACTION_RECEIVE`.

### Root-directory reorganization debate intensifies

Patrick Steinhardt refined his proposal to move libgit.a sources into a `lib/` directory, emphasizing discoverability for newcomers over code organization. Junio C Hamano countered that `lib/` is an arbitrary intermediate layer that fails to deliver meaningful organization, contrasting it with past subsystem moves tied to substantial updates, and warned the patch may worsen navigation. Steinhardt doubled down on the newcomer-focused rationale, conceding the change is subjective. The debate drew support from Johannes Schindelin and brian m. carlson for the bulk move, while SZEDER Gábor argued it trades one annoyance for another.

### `paint_down_to_common()` optimization series completes

Kristoffer Karlsson announced that v8 of the `paint_down_to_common()` optimization series is technically complete, fully reviewed, and queued in Junio's tree. Elijah Newren provided a `Reviewed-by` tag, and both Karlsson and Newren confirmed readiness for advancement to `next`.

### Windows-specific fixes and build system refinements

Johannes Schindelin posted v2 of the MinGW/Windows build adjustment series addressing cross-build-system inconsistencies, and a twelve-patch v2 series upstreaming Git for Windows build fixes including Python support and MSYS2 metadata handling. Johannes Sixt observed that some patches introduce variables used only in later patches. Separately, a textconv filter misconfiguration on Windows causing silent corruption of `git diff` output was traced to MSYS argument parsing mishandling `\r` escape sequences, with Jeff King confirming it as user misconfiguration rather than a Git bug.

## In brief

**`git repack --drop-filtered` ready for `next`** -- Siddharth Shrimali's v5 series for partial clone disk-space recovery addressed a type-mismatch bug and clarified deferred items, with Christian Couder, Junio C Hamano, and others confirming readiness.

**`worktree add` out-of-bounds read fix** -- Matthias Aßhauer's patch fixes an out-of-bounds read in `worktree_basename` when `path` is empty, preventing potential data loss; René Scharfe declined authorship in favor of Aßhauer.

**`fetch_if_missing` libification** -- Tian Yuchen's v6 series moves the global `fetch_if_missing` into `struct repository`, with Junio C Hamano reviewing favorably and asking about `git index-pack` standalone design intent.

**`format-rev` new options** -- Kristoffer Haugsbakk's series adds `--abbrev`, `--color`, and `--date` to `git format-rev`; Junio C Hamano caught an assertion-order bug in the date callback that could segfault.

**`rev-parse --output-object-format` cross-hash** -- Dimitri John Ledkov's patch allows computing object names in another hash algorithm on demand, refusing commits and trees with submodules.

**Eager ODB alternates loading** -- Patrick Steinhardt's series makes the object database eagerly load alternates at initialization; Jeff King questioned `core.ignorecase` for deduplication on different filesystems.

**Runtime `core.useNanosec`** -- D. Ben Knoble's v2 series converts build-time `USE_NSEC` to a runtime config; Patrick Steinhardt suggested pausing `repo_config_values()` conversions until a plan exists for non-`the_repository` repos.

**OCSP validation** -- graysongordon-gl's `http.sslVerifyStatus` patch enables validating stapled OCSP responses, with motivation citing government customers mandating OCSP stapling.

**`http.sslVerifyHost` opposed** -- Patrick Steinhardt and brian m. carlson argued the proposed option oversells security benefits and is vulnerable to active MITM attacks.

**Git for Windows security update** -- Johannes Schindelin announced Git for Windows 2.55.0(4), fixing CVE-2026-62960 in bundle URI handling.

**Bash completion fixes** -- Junio C Hamano's v6 series fixes `git checkout -C` completion to suggest only tracked paths; Matthew Hughes reported `git add` completion hanging in large repos and proposed a `GIT_COMPLETION_NO_COMPLETE_INDEX` escape hatch.

**`git squash` command support** -- Matej Dujava and Bradley Morgan expressed support for adding a `git squash` command, with Dujava suggesting `fixup!`/`squash!` workflows.

**`bitmapPseudoMerge.*` documentation** -- kapicharly@icloud.com posted documentation for experimental pseudo-merge bitmap configuration options.

**FSEvents queue regression** -- Tamir Duberstein traced the regression to an April 2026 timeout change, citing Apple documentation that FSEvents queue drainage is non-deterministic.

**Valueless config warnings** -- Tilak Raaz posted v4 making valueless `submodule.active` emit a warning via `config_error_nonbool()`; Elijah Newren posted a bugfix rejecting valueless `promisor-remote` capability in protocol v2.

**Proxy authentication bug** -- Tom Noonan II reported `git-remote-https` failing to extract credentials from proxy URLs.

**Worktree lifecycle hooks** -- Alexander G. Riccio described a use case for worktree lifecycle hooks citing orphaned Xcode build cache data.

**`worktree repair` relative paths** -- Yoichi NAKAYAMA fixed `worktree repair` to handle relative paths in `.git` files correctly.

**Reflink support** -- Vlad Petric posted a patch adding reflink support for local clones with revised fallback order.

**Segmentation fault fix** -- Christian Couder posted a bugfix preventing a segfault when `git --shallow-file` is invoked without an argument.

**Memory-safety fix** -- René Scharfe proposed a minimal rewrite of `worktree_basename()` to eliminate a pointer underflow.

**Coccinelle performance** -- Jeff King's patch to bump the Ubuntu image for static-analysis jobs was contested after SZEDER Gábor's benchmarks showed Coccinelle 1.3.1 as 4.5x slower than 1.1.1.

**GSoC updates** -- Siddharth Shrimali shared Week 11 progress on partial clone disk-space recovery and K Jayatheerth's `git repo info` path keys series saw continued review.

**`git send-email` bugfix** -- Harald Nordgren's fix for missing "Subject:" headers was approved for `next`.

**`git worktree add` error messages** -- Yoichi NAKAYAMA's v3 patch improving ambiguous remote branch error messages addressed Junio's feedback.

**`git maintenance` promisor packs** -- Taylor Blau's two-patch bugfix for geometric repacking was reviewed with a resolved edge case.

**`git history` Bash completion** -- Vincent Mailhol's v2 series added completion for the new command.

**Reflog API alternative** -- Patrick Steinhardt proposed extending reference transactions to handle reflog edits natively instead of a new bulk API.

**Trace2 instrumentation** -- Patrick Steinhardt approved test extensions for `git fetch-pack` instrumentation.

**Coccinelle rules** -- Patrick Steinhardt distinguished between temporary migration rules and permanent antipattern enforcement.

**TUI proposal** -- Steinhardt refined his proposal to focus on targeted UI improvements rather than a full UI.

**Gitk color dialog** -- Johannes Sixt posted v2 of the color-preference dialog overhaul with six patches consolidating logic and improving usability.

**Flaky test fix** -- Patrick Steinhardt posted v2 refactoring tests to use throwaway repositories.

**`git diff -l` help text** -- Elijah Newren proposed revised wording; Junio suggested making it more user-friendly.

**Notes in `git log --all`** -- Marcel Svitalský reported unexpected pseudo-commits from Git notes; D. Ben Knoble proposed workarounds.

**Midx utility function** -- Patrick Steinhardt and Taylor Blau agreed on placing a general utility in `lib-midx.sh` despite awkwardness.

**Performance patch withdrawn** -- Tamir Duberstein withdrew a patch after failing to prove it resolved the reported issue.

**`git worktree add` copy-on-write** -- Peter Morris proposed filesystem copy-on-write; Junio sketched an approach but declined over TOCTOU concerns.

**Filesystem cloning RFC** -- brian m. carlson cautioned against ReFS support citing corruption risks; Peter Morris countered that `CopyFile` and `FICLONE` avoid the issue.

**`chdir-notify` cleanup** -- Jeff King affirmed Colin Hinton's patch dropping the unused `name` parameter.

## Looking ahead

The `git repack --drop-filtered` series and the promisor object filtering series are poised to advance to `next` following their readiness declarations. Justin Tobler's ODB transaction work will see follow-ups adding `ODB_TRANSACTION_FETCH` for `git fetch-pack` and addressing `.keep` file design questions. The root-directory reorganization debate and the `uploadpack.lazyFetchTrusted` series remain in flight, with Junio C Hamano's proposed callback-based API awaiting response. The `git rebase` submodule regression awaits confirmation on `master` and a minimal reproducer.