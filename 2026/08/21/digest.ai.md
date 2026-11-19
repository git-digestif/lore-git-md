# Git mailing list daily digest for 2026/08/21

## Long threads

### Pluggable packfile writes for `git-receive-pack(1)`

*Long thread (root 2026/08/06/21-38-53): 35 replies so far from 3 participants over 16 days; 2 messages that day.*

This series in `builtin/receive-pack.c` and `odb/transaction.c` introduces a generic `odb_transaction_write_pack()` transaction interface for writing incoming packfiles, with the existing logic that spawns `git-index-pack(1)` and `git-unpack-objects(1)` becoming the "files" backend implementation. Patrick Steinhardt said he is happy with the v5 series. Junio C Hamano observed that reading `receive.unpackLimit` and `transfer.unpackLimit` in generic object-layer code feels like a layering violation, explained that the decision whether to unpack or index is inherently up to the file backend, and attributed the current `*.unpackLimit` names to a historical wart, suggesting `odb-file.unpackLimit` would have been more accurate today.

### Pluggable packfile generation in the object database

*Long thread (root 2026/08/07/10-45-06): 31 replies so far from 6 participants over 15 days; 8 messages that day.*

Patrick Steinhardt's series makes packfile generation pluggable in the ODB so the transport layer can generate packfiles through a generic interface rather than only via the "files" backend, hoisting the machinery shared by upload-pack, send-pack, and bundle. He posted v4, whose changelog lists an improved error message and a small stylistic fix. Elijah Newren reviewed the cover letter, said he likes the design, and confirmed the 1/6 fix addresses the use-after-scope bug he raised on v2. Patrick Steinhardt answered Junio C Hamano's question that `opts->pack_fd` is consumed by the generator and that a primary source lacking the `generate_pack` callback would die, since the callback is non-optional. Karthik Nayak reviewed v4 and stated "Everything looks good now. Thanks!"

### Eagerly loading ODB alternates

*Long thread (root 2026/08/10/13-33-27): 31 replies so far from 5 participants over 12 days; 2 messages that day.*

This series in the object database code loads alternates eagerly at ODB initialization instead of lazily on first access, removing scattered `odb_prepare_alternates()` calls, and decouples source path comparisons from `the_repository`. Patrick Steinhardt argued the alternate-deduplication mechanism is imperfect but has worked in practice for roughly twenty years with no reported problems, confirmed both alternate sources go through `parse_alternates()`, which makes paths absolute via `realpath()` without producing a canonical representation, and declined to declare that repositories cannot span filesystems with different case sensitivities. He confirmed `o->repo` is fully initialized when the ODB is set up. Junio C Hamano agreed the hashing mechanism should stay and proposed a best-effort case-insensitive match when the platform hints at a case-insensitive filesystem, concluding the code can stay as-is since inconsistent path casing would merely cause a redundant extra search of a duplicate alternate.

### New path keys for `git repo info`

*Long thread (root 2026/07/16/01-21-31): 25 replies so far from 4 participants over 37 days; 1 message that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info` in `builtin/repo.c`, including `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`, each with absolute and relative format suffixes. K Jayatheerth agreed that `path.git-prefix` is repository information complementing `cdup`, and committed to adding `cdup` in the next version.

### Ambiguous remote branch message for `worktree add`

*Long thread (root 2026/08/08/08-21-41): 24 replies so far from 6 participants over 14 days; 3 messages that day.*

Yoichi Nakayama's series improves the error and advice shown by `git worktree add` and `git checkout` when a branch name matches multiple remote tracking branches, listing the matched remotes via a shared `advise_disambiguating_remotes()` function. Junio C Hamano flagged a subject-verb agreement error in the new advice message and asked whether the one-argument `git worktree add` path should be disambiguated the same way. Yoichi Nakayama acknowledged the grammar issue, proposed rewording the advice to "Branch name '%s' appears in multiple remotes:", and explained that with `worktree.guessRemote` a multiple match still lets the command succeed by creating the branch from HEAD, so he does not think a warning is necessary there. Junio C Hamano accepted the current behavior as not regressing anything.

## The day in brief
Patrick Steinhardt declared himself happy with v5 of the pluggable packfile-writes series for `git-receive-pack(1)`, while Karthik Nayak gave v4 of the pluggable packfile-generation series a final "Everything looks good now." A new two-patch series adds parallel fetching of packfile URIs with a `fetch.packfileURIThreads` option, and the trailers patch that stops recognizing URLs as trailers was marked for `next`.

## Notable threads

### Trailers stop recognizing URLs as trailers

Kristoffer Haugsbakk's patch to `trailer.c` makes trailer parsing fail when a separator is followed by `://` before any whitespace, so HTTPS URLs at the end of commit messages are no longer mistaken for trailers. He posted v3, whose only change from v2 is adding "Acked-by: Jeff King" to the commit message after Jeff King said v2 looked great. Junio C Hamano confirmed the v3 change is the only one and marked the topic for `next`.

### Parallelizing packfile URI fetches in `fetch-pack`
Patrick Steinhardt submitted a two-patch series preparing `git-fetch(1)` and `git-clone(1)` to fetch packfile URIs in parallel, refactoring the logic in `fetch-pack.c` into preparation, fetch, and aggregation phases and adding the `fetch.packfileURIThreads` configuration option, which defaults to 1 (sequential) but supports 0 (auto-detect CPUs) or larger values. Benchmarks in the second patch show speedups of up to 7.58x on a throttled 2MB/s connection and 5.35x on an unthrottled connection when the server announces multiple packfiles.

### Worktree repair detects relative paths in `.git` files
Yoichi Nakayama's patch fixes `git worktree repair` so that relative paths recorded in a worktree's `.git` file are detected correctly, introducing `read_gitfile_raw()` which returns the path as-is without resolving it, unlike `read_gitfile_gently()`. He posted v3 adopting the `read_gitfile_raw(struct strbuf *contents, const char *path)` signature Junio C Hamano had proposed, with `read_gitfile_gently()` refactored to call it. Junio C Hamano initially questioned whether the split dropped needed relative-path resolution and `is_git_directory()` checks for the worktree caller, then retracted half of that objection, concluding the `setup.c` change is fine as is.

### Avoiding whole-tree pushes from shallow clones in `send-pack`
Elijah Newren submitted a patch adding reachable shallow grafts as negative pack tips so that pushing from a shallow clone no longer sends the entire toplevel tree, controlled by a new `push.shallowExcludeBoundary` config (default true) with an opt-out for `receive.shallowUpdate` receivers. Patrick Steinhardt reviewed it, questioning whether amending a shallow graft could send insufficient information and whether tree-level negotiation could verify the grafted tree, and Elijah Newren replied that amending a shallow graft creates a new root commit, that tree negotiation would arrive too late without `push.negotiate`, and that the manual walk is required to stop at peer-known commits and detect graft boundaries.

### Report hook for `git-receive-pack(1)`
Karthik Nayak's series introduces a report hook that lets the server intercept and modify the status report before it is sent to the client, needed for custom post-reference-update logic. He posted v2 with more verbose documentation and an extended test. Patrick Steinhardt suggested handling a non-zero hook exit like the pre-receive hook by marking all refs as failed instead of dying, and Junio C Hamano agreed that a failed report hook should explicitly tell the client the push was declined rather than letting the client assume no news is bad news.

## In brief
- Elijah Newren identified that the multi-pack-index race also affects `OBJECT_INFO_QUICK` callers such as upload-pack's object-existence checks, which get no second read, and plans a patch forcing a packfile reload even under `OBJECT_INFO_QUICK`; he also found a hole in his own unsubmitted fix where a vanished `.pack` with an already-mapped `.idx` would not trigger recovery.
- Volodymyr Vriukalo posted a two-patch series fixing `git branch --recurse-submodules` when the start point names no ref, which previously aborted with a BUG after already moving the branch; the fix guards the `setup_tracking()` call and makes the submodule helper's start-name argument optional.
- Alexey Samsonov posted a three-patch series migrating the codebase from the POSIX.1-2024-removed `utime()` to `utimensat()` with a `utimes()` fallback and a MinGW wrapper; Junio C Hamano noted macOS has supported `utimensat` since 10.13, so the migration is safe on Apple platforms.
- Patrick Steinhardt's patch raising the default `maintenance.geometric-repack.auto` loose-object threshold from 100 to 6700 drew a "Looks good to me" from Elijah Newren, who confirmed two objects in `objects/17/` suffice to trigger a repack because the limit rounds up to a multiple of 256.
- Friel's patch adding a `write_pack_file/wrote_bytes` Trace2 datum to `builtin/pack-objects.c` received "looks good to me" verdicts from both Jeff King and Junio C Hamano.
- Artur Bieniek's `git pull --hard` patch drew a response from Junio C Hamano arguing that `--ff-only` already covers the rewritten-upstream case, since a rewound upstream should be surfaced loudly rather than blindly followed.
- Lutz Lengemann's zsh completion patch for `git -C <path>` drew a discussion where D. Ben Knoble and Junio C Hamano clarified that empty `case` arms are valid under the POSIX grammar in Section 2.10.
