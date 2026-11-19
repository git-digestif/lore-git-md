# Git mailing list daily digest for 2026/08/21

## Long threads

### Pluggable packfile writes in `builtin/receive-pack`

*Long thread: 35 replies so far from 3 participants over 16 days; 2 messages that day.*

This series adds a generic `odb_transaction_write_pack()` interface so `git-receive-pack(1)` writes incoming packfiles through the ODB transaction layer rather than child processes tied to the "files" backend, which becomes the backend implementation. Patrick Steinhardt said he is happy with the v5 series. Junio C Hamano called reading `receive.unpackLimit` and `transfer.unpackLimit` in generic object-layer code a layering violation because these settings belong to the transfer layer, and suggested `odb-file.unpackLimit` as a more accurate name.

### Pluggable packfile generation in the object database

*Long thread: 31 replies so far from 6 participants over 15 days; 8 messages that day.*

This series makes packfile generation pluggable for the transport layer by hoisting the pack-generation machinery shared by upload-pack, send-pack and bundle into a new ODB interface. Patrick Steinhardt posted v4 with an improved error message and a stylistic fix, and confirmed `opts->pack_fd` is consumed by the generator and that a primary source without the `generate_pack` callback would die. Karthik Nayak reviewed v4 and said everything looks good. Elijah Newren, who found the earlier use-after-scope bug via AI-assisted review, liked the design and proposed an alternative for the `the_repository` cleanup in `bundle.c`, which Patrick defended.

### Eagerly loading ODB alternates

*Long thread: 31 replies so far from 5 participants over 12 days; 2 messages that day.*

This series makes the object database eagerly initialize alternates instead of scattering deferred `odb_prepare_alternates()` calls, and decouples source path comparisons from `the_repository`. Patrick Steinhardt argued the alternate-deduplication mechanism is imperfect but has worked for roughly 20 years, confirmed `realpath()` still gives no canonical representation, and declined to declare that repositories cannot span filesystems with different case sensitivities. Junio C Hamano agreed the hashing should stay, proposed a best-effort case-insensitive match where the platform hints at one, and concluded the code can stay as-is since inconsistent casing only causes a redundant duplicate search.

### New path keys for `git repo info`

*Long thread: 25 replies so far from 4 participants over 37 days; 1 message that day.*

This GSoC series adds `path.*` keys to `git repo info` for repository component paths -- toplevel, superproject root, hooks, index, grafts, git-prefix -- each with absolute and relative suffixes. K Jayatheerth agreed `path.git-prefix` is repository information complementing `cdup` and committed to adding a `cdup` key in the next version.

### Ambiguous remote branch message in `worktree add`

*Long thread: 24 replies so far from 6 participants over 14 days; 3 messages that day.*

This series improves the error and advice when `git worktree add` or `git checkout` matches multiple remote tracking branches with the same name, listing the matched remotes. Junio C Hamano flagged a grammar error in the advice and asked about the one-argument `worktree add` path. Yoichi Nakayama proposed rewording the advice to "Branch name '%s' appears in multiple remotes:" and explained that with `--guess-remote` a multiple match still succeeds by creating the branch from HEAD, so no warning is needed; Junio accepted the behavior as not regressing anything.

## The day in brief
Patrick Steinhardt's pluggable packfile generation series reached v4 with a clean review, and his new parallel packfile-URI fetch series introduced a `fetch.packfileURIThreads` option with benchmarked speedups. Kristoffer Haugsbakk's trailer fix for URLs was marked for `next`, and Elijah Newren's shallow-clone push optimization drew detailed review questions.

## Notable threads

### Parallel packfile URI fetches in `fetch-pack`
Patrick Steinhardt submitted a two-patch series letting `git-fetch(1)` and `git-clone(1)` fetch packfile URIs in parallel, refactoring `fetch-pack.c` into preparation, fetch and aggregation phases and adding the `fetch.packfileURIThreads` config (default 1, 0 for auto-detect), with benchmarks showing speedups up to 7.58x on a throttled connection.

### Stop recognizing URLs as trailers
Kristoffer Haugsbakk posted v3 of the patch that makes trailer parsing fail on `://` separators so HTTPS URLs at the end of commit messages are not mistaken for trailers; the only change from v2 is an added `Acked-by: Jeff King`. Junio C Hamano confirmed v3's single change and marked the topic for `next`.

### Avoid sending the whole tree when pushing from a shallow clone
Elijah Newren submitted a `send-pack` patch that adds reachable shallow grafts as negative pack tips so pushing from a shallow clone no longer sends the entire toplevel tree, controlled by a new `push.shallowExcludeBoundary` config (default true). Patrick Steinhardt reviewed it with questions about graft amendment, tree negotiation and `push.negotiate` defaults, which Elijah answered, agreeing to mark parameters `const` and combine the ref loops.

## In brief
- Yoichi NAKAYAMA posted v3 of the `worktree repair` fix for relative paths in `.git` files, adopting a caller-prepared `strbuf` signature for `read_gitfile_raw()`; Junio C Hamano initially questioned the split from `read_gitfile_gently()` then retracted half the objection.
- Karthik Nayak posted v2 of the report hook for `git-receive-pack(1)`, and Junio C Hamano agreed a failed report hook should explicitly tell the client the push was declined, modeled on pre-receive.
- Elijah Newren identified that the multi-pack-index race also affects `OBJECT_INFO_QUICK` callers like upload-pack's existence checks, and found a hole in his own fix when only the `.pack` vanishes.
- Volodymyr Vriukalo fixed `git branch --recurse-submodules` aborting with a BUG when the start point names no ref, guarding the tracking-name call and making the helper's start-name argument optional.
- Friel's `pack-objects` trace patch adding a `wrote_bytes` Trace2 datum got "looks good" verdicts from Jeff King and Junio C Hamano.
