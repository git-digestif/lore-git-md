# Git mailing list daily digest for 2026/08/14

## Long threads

### Moving libgit.a sources into a "lib/" directory

*Long thread: 37 replies so far from 12 participants over 121 days; 1 message that day.*

Patrick Steinhardt's RFC proposes moving all libgit.a sources from the repository root into a `lib/` directory, shrinking the top level from over 550 files to about 80 to help newcomers navigate. Earlier, SZEDER Gábor countered that the move trades one annoyance for another, while Junio C Hamano argued the rename fallout is an ongoing burden and proposed migrating one subsystem at a time; Johannes Schindelin and brian m. carlson defended the bulk move. Today, Michael Montalbo proposed a script-based grouping of the 231 libgit.a source files into directories, combining weighted commit-history area labels with a cscope call-graph check; it placed 163 files into 14 directories, left 68 ungrouped, and found 6 boundary groups spanning multiple directories.

### `git repack --drop-filtered` for partial clones

*Long thread: 35 replies so far from 4 participants over 30 days; 3 messages that day.*

Siddharth Shrimali's GSoC series adds `--drop-filtered` and `--dry-run` to `git repack` so partial clones can drop locally accumulated promisor blobs above a size threshold and rely on lazy fetch to restore them. Earlier, v5 fixed review feedback, and Shrimali clarified that remote verification, recency-based selection, and a drop log are follow-ups, not prerequisites. Today, Christian Couder reviewed v5 and said it looks good to him, Junio C Hamano asked whether to mark the topic for `next`, and Shrimali confirmed the series is ready from his side.

### Pluggable packfile writes for receive-pack

*Long thread: 28 replies so far from 3 participants over 9 days; 3 messages that day.*

Justin Tobler's series adds a generic `odb_transaction_write_pack()` interface so `git-receive-pack` writes incoming packfiles through the ODB transaction layer, with the existing `git-index-pack`/`git-unpack-objects` logic becoming the "files" backend. Earlier discussion centered on `.keep` file cleanup after transactions reordered the ODB source list. Today, Patrick Steinhardt noted the `write_pack()` callback needs to know which scenario it runs in; Tobler replied that flags like `ODB_TRANSACTION_RECEIVE` already exist for this, that he will add `ODB_TRANSACTION_FETCH` for `git-fetch-pack` in a follow-up, and that he will use the flags in the next version.

### Fixing the O(N^2) packfile-loading regression

*Long thread: 20 replies so far from 5 participants over 3 days; 4 messages that day.*

Johannes Schindelin's patch fixes a regression where `packfile_store_add_pack()` linearly scanned the pack list on every insertion, making loading N packs O(N^2) and slowing `git rev-parse --short HEAD` from 0.4s to 4.5s with 37,815 packs; it adds a `skip_dup_check` parameter to `packfile_list_append()`. Today, Patrick Steinhardt agreed a hashmap already exists via `packfile_store_add_pack()` and was happy with the v2 patch, Junio C Hamano marked it for `next`, and Jeff King countered that the map avoids all of `add_packed_git()` -- allocation and stat calls -- not just list insertion.

## The day in brief
The `git repack --drop-filtered` series for partial clones earned a positive review and a question about marking it for `next`. Johannes Schindelin's fix for the quadratic packfile-loading regression was marked for `next` after Jeff King noted the existing hashmap protects more than list insertion. Dimitri John Ledkov submitted a patch letting `git rev-parse --output-object-format` compute object names in another hash algorithm on demand.

## Notable threads

### `rev-parse --output-object-format` in another hash algorithm
Dimitri John Ledkov submitted a patch so `git rev-parse --output-object-format=<algo>` can answer in a different hash algorithm: when `extensions.compatObjectFormat` has no mapping, `repo_oid_to_algop()` reads the object, converts its contents, and hashes the result, caching names for the process lifetime. It refuses commits (converting one converts the whole history) and trees with submodules, naming the offending gitlink, and errors when a name cannot be produced instead of silently printing the storage name. On git.git's tree the computation takes 0.26s versus 0.25s by hand; on linux.git's 1.6GiB tree, 8s.

### Eagerly loading ODB alternates
Patrick Steinhardt's series makes the object database eagerly load alternates at initialization rather than on first lookup, removing scattered `odb_prepare_alternates()` calls. Today's discussion centered on `odb_source_paths_cmp()`: Jeff King questioned whether `core.ignorecase` is correct for deduplicating alternates on different filesystems, proposing always-case-sensitive comparisons; Junio C Hamano warned a duplicated alternate could endanger a future duplicate-removal tool and endorsed the simpler approach; Jeff King confirmed macOS as the concern behind the unicode normalization footnote.

### Runtime `core.useNanosec` replacing build-time USE_NSEC
D. Ben Knoble's v2 series converts the build-time `USE_NSEC` macro into a runtime `core.useNanosec` config (default false), replacing `#ifdef` paths in `is_racy_stat()` and `match_stat_data()` with runtime checks. Today, Patrick Steinhardt said Git should pause converting everything to `repo_config_values()` until there is a plan for repositories other than `the_repository`, but did not want to hold the series hostage. Junio C Hamano questioned what `ST_CTIME_NSEC()` yields on platforms without nanosecond support; Knoble replied the config exposes system capabilities and agreed force-disabling `use_nanosec` in `NO_NSEC` builds makes sense.

## In brief

- Junio C Hamano's review of Christian Couder's `uploadpack.lazyFetchTrusted` series proposed a callback-based `path_allowlist_apply()` API and asked the docs to state upfront that the variable is multi-valued.
- Matthew Hughes reported bash completion for `git add` hanging for seconds in a repo with 367,628 files across 58,404 directories, and proposed a `GIT_COMPLETION_NO_COMPLETE_INDEX` escape hatch.
- Peter Morris proposed `git worktree add` use filesystem copy-on-write where supported; Junio C Hamano sketched a `checkout_entry()` approach but declined over TOCTOU concerns.
- Tilak Raaz posted v2 warning on valueless `submodule.active` via `config_error_nonbool()`; Junio C Hamano suggested the command should fail rather than `continue`.
