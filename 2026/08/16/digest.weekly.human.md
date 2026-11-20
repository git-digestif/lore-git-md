## Long threads

### Moving libgit.a sources into a "lib/" directory

*Long thread: 37 replies so far from 12 participants over 121 days.*

Patrick Steinhardt's RFC proposes moving all libgit.a sources from the repository root into a `lib/` directory, shrinking the top level from over 550 files to about 80 to help newcomers navigate. Earlier, SZEDER Gábor countered that the move trades one annoyance for another, while Junio C Hamano argued the rename fallout is an ongoing burden and proposed migrating one subsystem at a time. During this period, Steinhardt refined the proposal to emphasize discoverability for newcomers, while Junio warned the `lib/` directory is an arbitrary intermediate layer that may worsen navigation. Michael Montalbo proposed a script-based grouping of the 231 libgit.a source files into 14 directories using commit-history labels and cscope call-graph checks, placing 163 files and leaving 68 ungrouped. The debate remained unresolved by week's end.

### `git repack --drop-filtered` for partial clones

*Long thread: 35 replies so far from 4 participants over 30 days.*

Siddharth Shrimali's GSoC series adds `--drop-filtered` and `--dry-run` to `git repack` so partial clones can drop locally accumulated promisor blobs above a size threshold and rely on lazy fetch to restore them. During the week, v4 and v5 addressed review feedback, including a type-mismatch bug in `list_objects_filter__filter_oidset()` where `unsigned long` was passed where `size_t *` was expected. Christian Couder reviewed v5 and approved it, Junio C Hamano asked whether to mark the topic for `next`, and Shrimali confirmed the series is ready, with remote verification, recency-based selection, and a drop log explicitly deferred as follow-up work.

### Pluggable packfile writes for receive-pack

*Long thread: 28 replies so far from 3 participants over 9 days.*

Justin Tobler's series adds a generic `odb_transaction_write_pack()` interface so `git-receive-pack` writes incoming packfiles through the ODB transaction layer, with the existing `git-index-pack`/`git-unpack-objects` logic becoming the "files" backend. During the week, v3 fixed a critical lockfile lifecycle flaw where `.keep` files were moved during commit but tempfile structures still pointed to pre-migration paths. Patrick Steinhardt noted the `write_pack()` callback needs to know which scenario it runs in; Tobler replied that flags like `ODB_TRANSACTION_RECEIVE` already exist, that he will add `ODB_TRANSACTION_FETCH` for `git-fetch-pack` in a follow-up, and that he will use the flags in the next version.

### `git repo info` path keys from the GSoC series

*Long thread: 21 replies so far from 4 participants over 32 days.*

K Jayatheerth's GSoC series adds new keys to `git repo info` that output the paths of repository components: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`, each supporting absolute and relative suffixes for formatting. During the week, Lucas Seiki Oshiro questioned why the test for `path.toplevel` formatting does not use the `test_repo_info_path` helper, and later objected that `path.git-prefix` seems out of place in `git repo info`, arguing it is current directory information rather than repository information, while acknowledging the information is relevant.

### Fixing the O(N^2) packfile-loading regression

*Long thread: 20 replies so far from 5 participants over 3 days.*

Johannes Schindelin's patch fixes a regression where `packfile_store_add_pack()` linearly scanned the pack list on every insertion, making loading N packs O(N^2) and slowing `git rev-parse --short HEAD` from 0.4s to 4.5s with 37,815 packs; it adds a `skip_dup_check` parameter to `packfile_list_append()`. During the week, Patrick Steinhardt agreed a hashmap already exists via `packfile_store_add_pack()` and approved the v2 patch, Junio C Hamano marked it for `next`, and Jeff King countered that the map avoids all of `add_packed_git()` -- allocation and stat calls -- not just list insertion.

## The period in brief

The week of 2026/08/10 -- 2026/08/16 saw heavy traffic across seven active days, with 81 emails on Monday alone. The tone was urgent around several regressions -- in `git rebase` with submodules, `git maintenance` geometric repacking, and a quadratic packfile-loading slowdown -- while long-running architectural efforts like the ODB abstraction and root-directory reorganization advanced through heated debate. Security concerns surfaced in `gitk` custom commands and OCSP validation, and multiple GSoC series reached readiness for integration.

## Key developments

### Regressions in rebase, maintenance, and packfile loading

Three regressions dominated the week's urgent traffic. A `git rebase` regression bisected to `bb5da75d61` caused fatal commit-graph errors with submodules, with Patrick Steinhardt investigating whether master already fixes it and requesting a minimal reproducer. Stefan Haller reported concurrency failures and an over-aggressive auto-condition in `git maintenance` geometric repacking, with Patrick Steinhardt proposing a threshold fix from 100 to 6700 to match `gc.auto`. Johannes Schindelin fixed an O(N^2) regression in `packfile_store_add_pack()` that slowed `git rev-parse` from 0.4s to 4.5s with many packs; Jeff King noted the existing hashmap protects more than list insertion, and Junio marked it for `next`.

### ODB abstraction advances on multiple fronts

Justin Tobler's series extending ODB transactions to `git receive-pack` reached v3, fixing a critical lockfile lifecycle flaw and adding `odb_transaction_write_pack()`. Patrick Steinhardt posted v2 of eagerly loading ODB alternates and questioned the `.keep` file design in the transaction system. Tian Yuchen moved `fetch_if_missing` into `struct repository` in v6, with Junio reviewing favorably and asking whether `git index-pack` documentation clarifies its standalone design intent. These steps advance Patrick Steinhardt's goal of backend-agnostic object storage.

### Root-directory reorganization debate

Patrick Steinhardt refined his proposal to move libgit.a sources into a `lib/` directory, arguing it improves discoverability for newcomers by making entry points like `README.md` and `Makefile` more visible. Junio C Hamano countered that `lib/` is an arbitrary intermediate layer that fails to deliver meaningful organization, contrasting it with past subsystem moves tied to substantial updates, and warned the patch may worsen navigation. Michael Montalbo proposed a script-based grouping of 231 files into 14 directories using commit history and call-graph analysis. The debate remained unresolved by week's end.

### Security fixes and features

Tim Wiederhake's `gitk` custom commands patch faced a critical shell-injection flaw via unquoted placeholder substitution, with Junio providing a concrete exploit example using `title?'; echo no'`. graysongordon-gl posted `http.sslVerifyStatus` for OCSP validation, citing government customers who mandate stapling. Johannes Schindelin announced Git for Windows 2.55.0(4) fixing CVE-2026-62960 in bundle URI handling. Matthias Aßhauer fixed a data-loss bug in `git worktree add ""` where an out-of-bounds read in `worktree_basename` could cause recursive deletion of the current working directory including `.git`.

### Feature series reach readiness

Kristofer Karlsson's v8 `paint_down_to_common()` optimization series was declared technically complete, fully reviewed, and queued in Junio's tree. Elijah Newren and Kristofer Karlsson approved the promisor object filtering series for `next`. Siddharth Shrimali's `git repack --drop-filtered` v5 addressed a type-mismatch bug and received Christian Couder's approval, with Junio asking whether to mark it for `next`.

### Windows and build system work

Johannes Schindelin posted v2 of MinGW/Windows build adjustments and a twelve-patch series upstreaming Git for Windows fixes, with Johannes Sixt noting that `MINGW_PREFIX` compile definitions in patch 08/12 do not belong there yet since only patch 12/12 uses the variable. D. Ben Knoble's v2 converted build-time `USE_NSEC` to runtime `core.useNanosec`. Jeff King's CI patch to bump the Ubuntu image was contested after SZEDER Gábor showed Coccinelle 1.3.1 is 4.5x slower than 1.1.1.

## In brief

- **`git send-email`:** Harald Nordgren's bugfix for missing "Subject:" headers was approved for `next`.
- **`git worktree add`:** Yoichi NAKAYAMA's v3 improved error messages for ambiguous remote branches.
- **`git history` completion:** Vincent Mailhol's v2 added Bash completion with D. Ben Knoble advocating simplification.
- **GSoC updates:** Siddharth Shrimali shared Week 11 progress on partial clone disk-space recovery.
- **FSEvents regression:** Tamir Duberstein explained the queue behavior traced to the April 2026 timeout change.
- **Textconv on Windows:** Skybuck Flying reported MSYS argument parsing mishandling `\r` in sed commands.
- **Reflog API:** Patrick Steinhardt proposed extending reference transactions for reflog edits instead of a new API.
- **Trace2:** Patrick Steinhardt approved instrumentation for `git fetch-pack`.
- **Memory safety:** René Scharfe proposed rewriting `worktree_basename()` to fix a pointer underflow.
- **Coccinelle:** Patrick Steinhardt distinguished temporary migration rules from permanent antipattern enforcement.
- **TUI proposal:** Steinhardt refined to targeted UI improvements rather than a full UI.
- **Segfault fix:** Christian Couder fixed `git --shallow-file` without argument.
- **`http.sslVerifyHost`:** Steinhardt and brian m. carlson opposed the option as vulnerable to MITM.
- **Reflink:** Vlad Petric added reflink support for local clones.
- **Worktree lifecycle hooks:** Alexander G. Riccio described orphaned Xcode build cache use case.
- **Proxy auth:** Tom Noonan II reported `git-remote-https` failing to extract proxy credentials.
- **Bash completion:** Junio fixed `git checkout -C` to suggest only tracked paths; Matthew Hughes reported hangs in large repos with 367,628 files.
- **`rev-parse --output-object-format`:** Dimitri John Ledkov added cross-hash computation on demand.
- **`uploadpack.lazyFetchTrusted`:** Junio proposed callback-based API for Christian Couder's series.
- **`submodule.active`:** Tilak Raaz posted v4 warning on valueless config.
- **`format-rev`:** Kristoffer Haugsbakk added `--abbrev`, `--color`, `--date` with Junio catching an assertion-order bug.
- **`bitmapPseudoMerge.*`:** kapicharly documented experimental config options.
- **`git squash`:** Matej Dujava and Bradley Morgan supported the proposed command.

## Looking ahead

The `git repack --drop-filtered` series and promisor object filtering series are poised for `next` pending final confirmation. The root-directory reorganization debate will likely continue as Patrick Steinhardt and Junio C Hamano seek consensus. Justin Tobler plans to add `ODB_TRANSACTION_FETCH` flags for `git-fetch-pack` in a follow-up to the receive-pack transaction series.