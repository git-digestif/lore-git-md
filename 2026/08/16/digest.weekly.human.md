## Long threads

### Moving libgit.a sources into a "lib/" directory
*Long thread: 37 replies so far from 12 participants over 121 days.*
Patrick Steinhardt's RFC proposes moving all libgit.a sources from the repository root into a `lib/` directory to shrink the top level from over 550 files to about 80, aiding newcomer navigation. The debate has drawn contrasting views: SZEDER Gábor argued the move trades one annoyance for another, while Junio C Hamano contended the `lib/` directory is an arbitrary intermediate layer that fails to deliver meaningful organization and warned it may worsen navigation. Johannes Schindelin and brian m. carlson defended the bulk move. Michael Montalbo recently proposed a script-based grouping of the 231 libgit.a source files into 14 directories using commit-history area labels and cscope call-graph checks, placing 163 files and leaving 68 ungrouped. The thread remains active with no resolution in sight.

### `git repack --drop-filtered` for partial clones
*Long thread: 35 replies so far from 4 participants over 30 days.*
Siddharth Shrimali's GSoC series adds `--drop-filtered` and `--dry-run` to `git repack`, allowing partial clones to drop locally accumulated promisor blobs above a size threshold and rely on lazy fetch to restore them. After several iterations addressing review feedback, v5 fixed a type-mismatch bug identified by Junio C Hamano in `list_objects_filter__filter_oidset()` where an `unsigned long` was passed where `size_t` was expected. Christian Couder reviewed v5 positively, and Junio asked whether to mark the topic for `next`; Shrimali confirmed readiness. Remote verification, recency-based selection, and a drop log are explicitly deferred as follow-up work.

### Pluggable packfile writes for receive-pack
*Long thread: 28 replies so far from 3 participants over 9 days.*
Justin Tobler's series adds a generic `odb_transaction_write_pack()` interface so `git-receive-pack` writes incoming packfiles through the ODB transaction layer, with existing `git-index-pack`/`git-unpack-objects` logic becoming the "files" backend. Earlier discussion centered on `.keep` file cleanup after transactions reordered the ODB source list. Patrick Steinhardt noted the `write_pack()` callback needs to know which scenario it runs in; Tobler replied that flags like `ODB_TRANSACTION_RECEIVE` already exist, that he will add `ODB_TRANSACTION_FETCH` for `git-fetch-pack` in a follow-up, and that he will use the flags in the next version.

### `git repo info` path keys series
*Long thread: 21 replies so far from 4 participants over 32 days.*
K Jayatheerth's GSoC series adds path keys to `git repo info` outputting locations of repository components: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`, each supporting absolute and relative format suffixes. Earlier revisions renamed `superproject-working-tree` to `superproject-root`, dropped `path.objects` until pluggable ODB backends arrive, and moved the `prefix` member into `struct repository`. Lucas Seiki Oshiro recently questioned whether `path.git-prefix` belongs in `git repo info`, arguing it is current directory information rather than repository information, and asked why certain tests do not use the `test_repo_info_path` helper.

### Fixing the O(N^2) packfile-loading regression
*Long thread: 20 replies so far from 5 participants over 3 days.*
Johannes Schindelin's patch fixes a regression where `packfile_store_add_pack()` linearly scanned the pack list on every insertion, making loading N packs O(N^2) and slowing `git rev-parse --short HEAD` from 0.4s to 4.5s with 37,815 packs. The fix adds a `skip_dup_check` parameter to `packfile_list_append()`. Patrick Steinhardt agreed a hashmap already exists via `packfile_store_add_pack()` and approved the v2 patch; Junio C Hamano marked it for `next`. Jeff King countered that the map avoids all of `add_packed_git()` -- allocation and stat calls -- not just list insertion.

## The period in brief

The week of 2026/08/10 through 2026/08/16 saw seven active days of steady traffic, marked by urgent regression fixes, security discussions, and continued progress on the ODB abstraction. Readers should not miss the `git rebase` submodule regression, the `gitk` security flaw, and the advancement of the `git repack --drop-filtered` series toward `next`.

## Key developments

### Regressions and security fixes
Three urgent issues dominated the week. A `git rebase` regression bisected to `bb5da75d61` causes fatal errors with submodules and commit graphs, with Patrick Steinhardt asking for confirmation on `master` and a minimal reproducer. Stefan Haller reported two critical `git maintenance` geometric repacking issues: concurrency failures in `prune_packed_objects()` and an auto-condition bug triggering excessive repacks; Patrick Steinhardt proposed raising the default threshold from 100 to 6700. A critical security flaw in Tim Wiederhake's `gitk` custom commands patch allows arbitrary command injection via unquoted placeholder substitution, with Junio C Hamano providing a concrete exploit example; the author is evaluating fixes. Matthias Aßhauer's patch fixes an out-of-bounds read in `worktree_basename` that could make `git worktree add ""` recursively delete the current directory.

### ODB abstraction and receive-pack transactions
Justin Tobler advanced the ODB abstraction with v3 of a nine-patch series extending ODB transactions to `git receive-pack`, fixing a critical lockfile lifecycle flaw in the "files" backend and introducing `odb_transaction_write_pack()`. Patrick Steinhardt posted v2 of the ODB alternates series making the object database eagerly load alternates at initialization. Tian Yuchen posted v6 moving the global `fetch_if_missing` variable into `struct repository`, which Junio C Hamano reviewed favorably.

### Series advancing toward integration
Several long-running series reached milestones. Kristofer Karlsson's v8 `paint_down_to_common()` optimization series was declared technically complete and queued in Junio's tree. Elijah Newren and Kristofer Karlsson approved the v8 promisor object filtering series, with Junio asking whether to mark it for `next`. Siddharth Shrimali's `git repack --drop-filtered` series reached v5 with all feedback addressed and was confirmed ready for `next`. Johannes Schindelin's fix for the O(N^2) packfile-loading regression was marked for `next`.

### Root-directory reorganization debate
Patrick Steinhardt refined his proposal to move libgit.a sources into a `lib/` directory, emphasizing discoverability for newcomers. Junio C Hamano countered that `lib/` is an arbitrary intermediate layer that fails to deliver meaningful organization, warning the patch may worsen navigation. The debate continued with Michael Montalbo proposing a script-based grouping approach.

## In brief

- **`git send-email`:** Harald Nordgren's bugfix for missing "Subject:" headers was approved and marked for `next`.
- **`git worktree add`:** Yoichi NAKAYAMA's v3 patch improving error messages for ambiguous remote branches addressed Junio's feedback.
- **`git maintenance` promisor packs:** Taylor Blau's two-patch bugfix for geometric repacking was reviewed by Patrick Steinhardt.
- **`git history` Bash completion:** Vincent Mailhol's v2 series added completion for the new command.
- **GSoC updates:** Progress shared on partial clone disk-space recovery and `gc-promisor` maintenance.
- **CI/build system:** Jeff King's patch to bump the Ubuntu image was contested after SZEDER Gábor's benchmarks showed Coccinelle 1.3.1 as 4.5x slower than 1.1.1.
- **FSEvents queue regression:** Tamir Duberstein traced the regression to the April 2026 timeout change.
- **Textconv filter on Windows:** Skybuck Flying reported a misconfigured textconv filter causing silent corruption; Jeff King confirmed it as a shell-quoting problem.
- **Reflog API:** Patrick Steinhardt countered Junio's proposal with an alternative extending reference transactions.
- **Trace2 instrumentation:** Patrick Steinhardt approved the test extension for `git fetch-pack`.
- **Memory-safety fix:** René Scharfe proposed a rewrite of `worktree_basename()` to eliminate a pointer underflow.
- **Coccinelle role:** Patrick Steinhardt distinguished between temporary API-migration rules and permanent antipattern-enforcement rules.
- **TUI proposal:** Steinhardt refined his proposal to focus on targeted UI improvements.
- **Segfault fix:** Christian Couder posted a bugfix for `git --shallow-file` without a required argument.
- **`http.sslVerifyHost`:** Patrick Steinhardt and brian m. carlson opposed the new option as vulnerable to MITM attacks.
- **Reflink support:** Vlad Petric posted a patch adding reflink support for local clones.
- **OCSP validation:** graysongordon-gl posted a patch introducing `http.sslVerifyStatus` for OCSP stapling validation.
- **Git for Windows security update:** Johannes Schindelin announced Git for Windows 2.55.0(4), fixing CVE-2026-62960.
- **Worktree lifecycle hooks:** Alexander G. Riccio described a use case involving orphaned Xcode build cache data.
- **Proxy authentication bug:** Tom Noonan II reported a bug in `git-remote-https` failing to extract credentials from proxy URLs.
- **Bash completion:** Junio posted a two-patch series fixing completion for `git checkout -C <dir>`.
- **`rev-parse --output-object-format`:** Dimitri John Ledkov submitted a patch to compute object names in another hash algorithm on demand.
- **Runtime `core.useNanosec`:** D. Ben Knoble's v2 series converts the build-time `USE_NSEC` macro to a runtime config.
- **`uploadpack.lazyFetchTrusted`:** Christian Couder's series remains stalled on a submission format issue.
- **`format-rev` options:** Kristoffer Haugsbakk's series adds `--abbrev`, `--color`, and `--date` to `git format-rev`.
- **Windows build refinements:** Johannes Schindelin posted v2 of MinGW/Windows build adjustments and a twelve-patch series upstreaming Git for Windows mingw patches.
- **Gitk color dialog:** Johannes Sixt posted v2 of the color-preference dialog overhaul.
- **`bitmapPseudoMerge.*` docs:** kapicharly@icloud.com posted documentation for the experimental configuration options.
- **`git squash`:** Matej Dujava and Bradley Morgan expressed support for adding a `git squash` command.
- **Midx utility function:** Patrick Steinhardt and Taylor Blau agreed placing a general utility function in `lib-midx.sh` was practical despite feeling odd.
- **Performance patch withdrawn:** Tamir Duberstein withdrew a patch after failing to prove it resolved the reported performance issue.
- **Topic integration:** Patrick Steinhardt asked Junio C Hamano to swap an existing topic for a newer one in `seen`; Junio agreed.
- **Documentation timing:** Kristoffer Haugsbakk announced plans to wait for the `kh/doc-trailers` topic to reach `master` before sending a new version.
- **Valueless `promisor-remote` capability:** Elijah Newren posted a bugfix rejecting valueless `promisor-remote` capability in Git protocol v2.
- **`git diff -l` help text:** Elijah Newren proposed revised wording, with Junio suggesting more user-friendly phrasing.
- **Notes in `git log --all`:** Marcel Svitalský reported unexpected pseudo-commits in a custom alias output, identified as Git notes; D. Ben Knoble proposed workarounds.
- **Bash completion hang:** Matthew Hughes reported `git add` completion hanging in a repo with 367,628 files and proposed a `GIT_COMPLETION_NO_COMPLETE_INDEX` escape hatch.
- **`git worktree add` filesystem copy:** Peter Morris proposed using copy-on-write where supported; Junio sketched an approach but declined over TOCTOU concerns.
- **`submodule.active` warning:** Tilak Raaz posted v2/v4 warning on valueless `submodule.active` via `config_error_nonbool()`.
- **`worktree repair` fix:** Yoichi NAKAYAMA submitted a patch handling relative paths in `.git` files correctly.
- **`chdir-notify` cleanup:** Jeff King affirmed Colin Hinton's patch dropping the unused `name` parameter.