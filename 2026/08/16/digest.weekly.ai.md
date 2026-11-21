## Long threads

### Moving libgit.a sources into a "lib/" directory

*Long thread: 37 replies so far from 12 participants over 121 days.*

Patrick Steinhardt's RFC proposes moving all libgit.a sources from the repository root into a `lib/` directory, shrinking the top level from over 550 files to about 80 to help newcomers navigate. Junio C Hamano countered that `lib/` is an arbitrary intermediate layer that fails to deliver meaningful organization and warned the rename fallout is an ongoing burden, while SZEDER Gábor argued the move trades one annoyance for another. Michael Montalbo proposed a script-based grouping of the 231 libgit.a source files into 14 directories using weighted commit-history labels and a cscope call-graph check, placing 163 files and leaving 68 ungrouped. The debate over whether the reorganization aids discoverability or merely obscures structure remains unresolved.

### `git repack --drop-filtered` for partial clones

*Long thread: 35 replies so far from 4 participants over 30 days.*

Siddharth Shrimali's GSoC series adds `--drop-filtered` and `--dry-run` to `git repack` so partial clones can drop locally accumulated promisor blobs above a size threshold and rely on lazy fetch to restore them. After v4 addressed all prior feedback, v5 fixed a type-mismatch bug where `list_objects_filter__filter_oidset()` passed an `unsigned long` address to a `size_t *` parameter. Christian Couder reviewed v5 positively, Junio C Hamano asked whether to mark the topic for `next`, and Shrimali confirmed the series is ready. Remote verification, recency-based culling, and a drop log are explicitly deferred as follow-up work rather than blocking requirements.

### Pluggable packfile writes for receive-pack

*Long thread: 28 replies so far from 3 participants over 9 days.*

Justin Tobler's series adds a generic `odb_transaction_write_pack()` interface so `git-receive-pack` writes incoming packfiles through the ODB transaction layer, with the existing `git-index-pack`/`git-unpack-objects` logic becoming the "files" backend. Earlier discussion centered on `.keep` file cleanup after transactions reordered the ODB source list. Patrick Steinhardt noted the `write_pack()` callback needs to know which scenario it runs in; Tobler replied that flags like `ODB_TRANSACTION_RECEIVE` already exist, that he will add `ODB_TRANSACTION_FETCH` for `git-fetch-pack` in a follow-up, and that he will use the flags in the next version.

### `git repo info` path keys from the GSoC series

*Long thread: 21 replies so far from 4 participants over 32 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info`, outputting the locations of repository components: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`, each supporting absolute and relative format suffixes. Earlier revisions renamed `superproject-working-tree` to `superproject-root`, dropped `path.objects` until pluggable ODB backends arrive, and moved the `prefix` member into `struct repository`. Lucas Seiki Oshiro objected that `path.git-prefix` seems out of place as "current directory information" rather than "repository information", and questioned why the `path.toplevel` test does not use the `test_repo_info_path` helper.

### Fixing the O(N^2) packfile-loading regression

*Long thread: 20 replies so far from 5 participants over 3 days.*

Johannes Schindelin's patch fixes a regression where `packfile_store_add_pack()` linearly scanned the pack list on every insertion, making loading N packs O(N^2) and slowing `git rev-parse --short HEAD` from 0.4s to 4.5s with 37,815 packs. The fix adds a `skip_dup_check` parameter to `packfile_list_append()`. Patrick Steinhardt agreed a hashmap already exists via `packfile_store_add_pack()` and approved the v2 patch, Junio C Hamano marked it for `next`, and Jeff King countered that the map avoids all of `add_packed_git()` -- allocation and stat calls -- not just list insertion.

## The period in brief

The week of 2026/08/10 through 2026/08/16 saw heavy traffic across seven active days, with 81 emails on Monday alone and sustained discussion through the weekend. The tone was urgent around several regressions affecting `git rebase`, `git maintenance`, and `git worktree add`, while long-running architectural efforts like the ODB abstraction and root-directory reorganization continued to generate debate. Readers should not miss the security flaw in `gitk` custom commands, the completion of the `paint_down_to_common()` optimization series, and the readiness of the `git repack --drop-filtered` series for integration.

## Key developments

### ODB abstraction advances across receive-pack and alternates

Justin Tobler posted v3 of a nine-patch series extending ODB transactions to `git receive-pack`, fixing a critical lockfile lifecycle flaw in the "files" backend where `.keep` lockfiles were moved during commit but tempfile structures still pointed to pre-migration paths. Patrick Steinhardt posted v2 of a series making the object database eagerly load alternates at initialization rather than on first lookup, removing scattered `odb_prepare_alternates()` calls. Both efforts advance Patrick Steinhardt's goal of backend-agnostic object storage, with Junio C Hamano reporting only minor merge conflicts during integration.

### Regression fixes for rebase, maintenance, and worktree

A regression in `git rebase` bisected to `bb5da75d61` causes fatal errors when submodules are involved because `lookup_commit_reference_gently()` resolves submodule OIDs against the main repository's commit graph; Patrick Steinhardt asked for confirmation whether the issue persists on `master`. Stefan Haller reported two critical issues in `git maintenance` geometric repacking: concurrency failures in `prune_packed_objects()` and an auto-condition bug triggering 256x more aggressively than documented, with Patrick Steinhardt proposing to raise the default threshold from 100 to 6700. Matthias Aßhauer fixed an out-of-bounds read in `worktree_basename` that could make `git worktree add ""` recursively delete the current working directory including `.git`.

### Root-directory reorganization debate continues

Patrick Steinhardt refined his proposal to move libgit.a sources into a `lib/` directory, emphasizing discoverability for newcomers over code organization. Junio C Hamano countered that the `lib/` directory is an arbitrary intermediate layer that fails to deliver meaningful organization, contrasting it with past subsystem moves tied to substantial updates, and warned the patch may worsen navigation by forcing contributors to re-learn the structure later. Steinhardt doubled down on the newcomer-focused rationale, conceding the change is subjective.

### `paint_down_to_common()` optimization series completes

Kristofer Karlsson announced that v8 of the `paint_down_to_common()` optimization series is technically complete, fully reviewed, and queued in Junio's tree. Elijah Newren provided a `Reviewed-by` tag and confirmed readiness, with Karlsson stating he would be very happy to have it advance to `next`. The series is now procedurally unblocked.

### Security and platform updates

Tim Wiederhake's `gitk` patch adding user-configurable custom commands contains a critical security flaw allowing arbitrary command injection via unquoted placeholder substitution; Junio C Hamano provided a concrete exploit example. graysongordon-gl posted v3 of a patch introducing `http.sslVerifyStatus` for OCSP validation during TLS handshakes, citing government customers who mandate OCSP stapling. Johannes Schindelin announced Git for Windows 2.55.0(4), fixing CVE-2026-62960 in bundle URI handling. Jeff King confirmed a Windows textconv filter misconfiguration was user error in MSYS argument parsing rather than a Git bug.

### `fetch_if_missing` moves into `struct repository`

Tian Yuchen posted v6 of a libification patch moving the global `fetch_if_missing` variable, which controls whether a missing-object check prompts a lazy fetch from a promisor remote, into `struct repository`, defaulting to 1 to preserve behavior. The series adjusts `builtin/fetch-pack.c`, `builtin/fsck.c`, `builtin/rev-list.c`, `builtin/index-pack.c`, and `builtin/pack-objects.c` so the `repo` pointer reaches every call site. Junio C Hamano reviewed the iteration favorably and asked whether running `git index-pack` outside a repository is legitimate or a user error.

## In brief

**`git send-email` bugfix** -- Harald Nordgren's fix for missing "Subject:" headers was approved and marked for `next`.

**`git worktree add` error messages** -- Yoichi NAKAYAMA's v3 patch improving error messages for ambiguous remote branches addressed Junio's feedback on commit-message clarity.

**`git history` Bash completion** -- Vincent Mailhol's v2 series added completion for the new `git history` command, with D. Ben Knoble advocating for a simplified approach.

**GSoC updates** -- Siddharth Shrimali shared Week 11 progress on partial clone disk-space recovery and a separate `gc-promisor` maintenance task.

**CI Coccinelle slowdown** -- Jeff King's patch to bump the Ubuntu image for static-analysis jobs was contested after SZEDER Gábor's benchmarks showed Coccinelle 1.3.1 as 4.5x slower than 1.1.1.

**Reflog API debate** -- Patrick Steinhardt countered Junio's proposal for `refs_reflog_edit_in_bulk()` with an alternative extending reference transactions to handle reflog edits natively.

**Trace2 instrumentation** -- Patrick Steinhardt approved a test extension for `git fetch-pack` instrumentation despite a minor test design quirk.

**Memory-safety fix** -- René Scharfe proposed a minimal rewrite of `worktree_basename()` to eliminate a pointer underflow.

**TUI proposal** -- Steinhardt refined his proposal to focus on targeted UI improvements rather than building a full UI.

**Segmentation fault fix** -- Christian Couder posted a bugfix preventing a segfault when `git --shallow-file` is invoked without a required argument.

**`http.sslVerifyHost` debated** -- Patrick Steinhardt and brian m. carlson opposed the option, arguing it oversells security benefits and is vulnerable to active MITM attacks.

**Reflink support** -- Vlad Petric posted a patch adding reflink support for local clones, revising the fallback order to hardlink, reflink, then byte-for-byte copy.

**Worktree lifecycle hooks** -- Alexander G. Riccio described a real-world use case citing gigabytes of orphaned build cache data left by Xcode.

**Proxy authentication bug** -- Tom Noonan II reported that `git-remote-https` fails to extract credentials from proxy URLs.

**Bash completion fixes** -- Junio posted a series fixing `git checkout -C <dir>` to only suggest tracked paths; Matthew Hughes reported `git add` completion hanging in a repo with 367,628 files and proposed a `GIT_COMPLETION_NO_COMPLETE_INDEX` escape hatch.

**ODB alternates v2** -- Patrick Steinhardt addressed minor feedback and clarified environment variable handling.

**Flaky test fix** -- Patrick Steinhardt posted v2 refactoring tests to use throwaway repositories and subshells.

**`.keep` file design** -- Steinhardt questioned the fundamental design of ".keep" files in the new transaction system.

**Valueless `promisor-remote` capability** -- Elijah Newren posted a bugfix rejecting valueless `promisor-remote` capability in Git protocol v2.

**`git diff -l` help text** -- Elijah Newren proposed revised wording; Junio suggested making it more user-friendly.

**Notes in `git log --all`** -- Marcel Svitalský reported unexpected pseudo-commits in a custom alias output, identified as Git notes.

**Performance patch withdrawn** -- Tamir Duberstein withdrew a patch after deeper investigation failed to prove it resolved the reported performance issue.

**`uploadpack.lazyFetchTrusted`** -- Junio's review proposed a callback-based `path_allowlist_apply()` API and asked docs to state the variable is multi-valued.

**Filesystem cloning for `git worktree add`** -- Peter Morris proposed copy-on-write; brian m. carlson cautioned against ReFS support citing corruption risks.

**Valueless `submodule.active`** -- Tilak Raaz posted v4 warning via `config_error_nonbool()` instead of a hard error.

**`worktree repair` relative paths** -- Yoichi NAKAYAMA fixed handling of relative paths in `.git` files.

**`format-rev` options** -- Kristoffer Haugsbakk's series adds `--abbrev`, `--color`, and `--date`; Junio caught an assertion-order bug that could segfault before the assertion fires.

**Windows build refinements** -- Johannes Schindelin posted v2 of a twelve-patch series upstreaming Git for Windows build fixes; Johannes Sixt noted `MINGW_PREFIX` usage timing.

**Gitk color dialog** -- Johannes Sixt posted v2 of a six-patch overhaul consolidating color-setting logic and making swatches clickable.

**`rev-parse --output-object-format`** -- Dimitri John Ledkov submitted a patch computing object names in another hash algorithm on demand.

**Runtime `core.useNanosec`** -- D. Ben Knoble's v2 converts the build-time `USE_NSEC` macro to a runtime config option.

**`bitmapPseudoMerge.*` docs** -- kapicharly@icloud.com posted documentation for the experimental configuration options.

**`git squash` command** -- Matej Dujava and Bradley Morgan expressed support, with Dujava suggesting `fixup!`/`squash!` workflows.