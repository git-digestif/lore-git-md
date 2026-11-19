# Git mailing list daily digest for 2026/08/14

## Long threads

### Moving libgit.a sources into a "lib/" directory

*Long thread (root 2026/04/16/13-24-29): 37 replies so far from 12 participants over 121 days; 1 message that day.*

Patrick Steinhardt's RFC series proposes moving all libgit.a sources into a separate `lib/` directory to shrink the top-level directory from over 550 files to about 80, making entry points like `README.md` and the build files more discoverable for newcomers. Earlier discussion weighed the one-time rename cost against the ongoing maintenance burden, with Junio C Hamano preferring piecemeal subsystem moves and Michael Montalbo previously proposing granular conceptual groupings scored by shared internal headers. Today Michael Montalbo proposed a script-based method to group the 231 libgit.a source files into directories, combining weighted commit-history area labels mapped through a 14-line CHARTER table with an independent cscope call-graph check; the script placed 163 files into 14 directories and left 68 ungrouped, and the call graph identified 6 boundary groups spanning multiple directories.

### repack --drop-filtered to reclaim space in partial clones

*Long thread (root 2026/07/16/13-28-41): 35 replies so far from 4 participants over 30 days; 3 messages that day.*

Siddharth Shrimali's GSoC series adds `git repack --drop-filtered` to reclaim space in partial clones by enumerating promisor blobs over a size threshold, dropping them locally, and relying on the existing lazy-fetch machinery to restore them on demand. Earlier rounds fixed an object-size type mismatch, made the filter options parameter const, and clarified that remote verification, recency-based selection, and a drop log are follow-ups after this series. Today Christian Couder reviewed v5 and stated "This version looks good to me"; Junio C Hamano asked whether he should mark the topic for `next`, and Siddharth Shrimali confirmed the series is ready from his side.

### Pluggable packfile writes for receive-pack

*Long thread (root 2026/08/06/21-38-53): 28 replies so far from 3 participants over 9 days; 3 messages that day.*

Justin Tobler's series makes `builtin/receive-pack` packfile writes pluggable via a generic `odb_transaction_write_pack()` interface, moving the `git-index-pack`/`git-unpack-objects` child-process logic into the "files" backend implementation, and fixes `.keep` files not being removed after ODB transactions reordered the source list. Today Patrick Steinhardt noted that since `write_pack()` is intended for more use cases than only `git-receive-pack`, the callback needs a way to be told which scenario it is running in; Justin Tobler replied that transaction flags such as `ODB_TRANSACTION_RECEIVE` already exist to differentiate callers, that he plans to add `ODB_TRANSACTION_FETCH` in a follow-up expanding the interface to `git-fetch-pack`, and that he will start using the flags in the next version.

### Packfile loading performance regression with many packs

*Long thread (root 2026/08/12/19-11-09): 20 replies so far from 5 participants over 3 days; 4 messages that day.*

Johannes Schindelin's patch fixes the O(N^2) regression introduced by 589127caa730, where `packfile_store_add_pack()` linearly scanned the pack list before every insertion, by adding a `skip_dup_check` parameter so known-new packs skip the scan; a reported case with 37,815 packs slowed `git rev-parse --short HEAD` from 0.4s to 4.5s. Today Patrick Steinhardt agreed a hashmap already exists via `packfile_store_add_pack()`/`packfile_store_load_pack()` and said he was happy with the v2 patch; Junio C Hamano marked it for `next`; Jeff King countered that the existing map avoids all of `add_packed_git()` -- allocation and `stat()` calls -- not just list insertion, so it could not simply become a check inside `packfile_list_append()`.

## The day in brief
The `repack --drop-filtered` series for partial clones earned a positive review and a readiness confirmation, with the maintainer asking about marking it for `next`. Johannes Schindelin's fix for the many-packfiles loading regression was marked for `next` after Jeff King refined the discussion about the existing hashmap. Dimitri John Ledkov submitted a new patch letting `rev-parse --output-object-format` compute object names in another hash algorithm on demand.

## Notable threads

### rev-parse computing object names in another hash algorithm
Dimitri John Ledkov submitted a patch enabling `git rev-parse --output-object-format=<algo>` to compute object names in another hash algorithm on demand: when `extensions.compatObjectFormat` has no recorded mapping for an existing object, `repo_oid_to_algop()` now reads the object, converts its contents, and hashes the result, caching computed names for the process lifetime. The patch refuses commits and trees containing submodules, accepts any hash algorithm git knows, and turns an unproducible name into an error instead of silently printing the storage name; on git.git's tree the computation takes 0.26s against 0.25s for reading and hashing by hand.

### Eagerly loading ODB alternates
Patrick Steinhardt's series makes the object database eagerly load alternates at initialization instead of deferring, removing `odb_prepare_alternates()` calls scattered across the codebase. Today Jeff King questioned whether `core.ignorecase` is strictly correct for alternate source path deduplication, since alternates may reside on different filesystems than the containing repository, and proposed always-case-sensitive comparisons or `stat()`-based confirmation; Junio C Hamano warned that a duplicated alternate entry could be dangerous for a future tool enumerating objects across alternates and endorsed the simpler approach; Jeff King confirmed macOS as the filesystem behind the unicode normalization concern.

### Converting build-time USE_NSEC to runtime core.useNanosec
D. Ben Knoble's series converts the build-time `USE_NSEC` into a runtime `core.useNanosec` config (default false) so distributed Git builds can avoid racy-git problems on supported filesystems. Today he posted v2, which reads the config through `struct repo` via `repo_config_values()`; Patrick Steinhardt cautioned against converting everything to `repo_config_values()` until there is a plan for repositories other than `the_repository`; Junio C Hamano flagged the `statinfo.c` change as iffy for a networked shared `$HOME/.gitconfig` across platforms, and Ben Knoble agreed that force-disabling `use_nanosec` in `NO_NSEC` builds makes sense.

### uploadpack.lazyFetchTrusted for server-side lazy fetch control
Christian Couder's series introduces `uploadpack.lazyFetchTrusted`, letting server operators mark repositories as trusted for lazy fetching when serving clones -- relevant when large-object promisors have moved blobs off the server. Today he apologized for a misthreaded v2 caused by passing `--in-reply-to` to `git format-patch`; Junio C Hamano reviewed the patches, proposing a callback-based `path_allowlist_apply()` API instead of a boolean `allow_dot` argument, and asking the docs to state upfront that the variable is multi-valued.

### Filesystem cloning for git worktree
Peter Morris proposed that `git worktree add` use filesystem copy-on-write or block cloning (e.g., ReFS block cloning on Windows Dev Drives) where supported, to cut SSD writes, physical disk usage, and creation time for large repositories used by concurrent AI agents. Today Kristoffer Haugsbakk pointed to an existing pull request covering Linux and macOS; Peter Morris argued Windows would get the benefit "for free" via the CopyFile API on the same ReFS volume; Junio C Hamano sketched an approach of intercepting `checkout_entry()` to CoW clean blob checkouts across linked worktrees with a post-copy hash against TOCTOU races, but said he is not interested in making such a change himself, mostly because of the TOCTOU issue.

## In brief
- Tian Yuchen posted v5 moving the global `fetch_if_missing` into `struct repository`; Junio C Hamano judged it "Much better" and suggested `OPT_CALLBACK` could carry the repo pointer.
- Tilak Raaz posted v2 of a GSoC patch warning on valueless `submodule.active` via `config_error_nonbool()`; Junio C Hamano questioned whether `continue` should instead fail the command.
- Matthew Hughes reported bash completion hanging in large repos (367,628 files across 58,404 directories) because `git ls-files` iterates the filesystem, and proposed a `GIT_COMPLETION_NO_COMPLETE_INDEX` workaround in `contrib/completion/git-completion.bash`.
- Patrick Steinhardt called Taylor Blau's bitmap-generation justification for the MIDX `--base` test "a bit roundabout", preferring direct verification that objects appear in the new layer.
- Patrick Steinhardt agreed to use `revs->repo` instead of `the_repository` in bundle packfile generation.
- **chdir-notify: drop unused `name` param (v2)**: Colin Hinton posted v2 removing the unused `name` parameter from chdir-notify APIs with an expanded commit message; Jeff King and Junio C Hamano called the patch good but asked for the history in the commit message.

- Kristoffer Haugsbakk confirmed the `[synopsis]` doc change was untested with AsciiDoc after a CI failure.
- Elijah Newren posted v2 clarifying the `-l` diff option help text; Junio C Hamano will mark it for `next`.
- Swapnil Saste fixed a typo in `Documentation/SubmittingPatches`; Junio C Hamano will apply it.
