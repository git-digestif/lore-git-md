# Git mailing list daily digest for 2026/10/20

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 31 replies so far from 4 participants over 32 days; 1 message that day.*

Harald Nordgren posted v10 of his five-patch series changing how `git fetch` behaves for a newly added remote in a shallow repository, so a plain fetch does not negotiate history for every branch the remote has; the v10 changelog says it resolves a merge conflict after master integrated `ch/fetch-followremotehead-lazy-validation`. The series adds a per-remote `remote.<name>.refmap` config as the default for `--refmap`, extracts the current-branch upstream check from `do_fetch()` into a `collect_upstream_from_remote()` helper in `remote.c`, makes a refspec-less `git fetch <name>` infer what to fetch from local branches whose `@{upstream}` is on that remote, adds `--limited-fetch` to `git remote add`, and defaults to it in already-shallow repositories. Earlier, Junio C Hamano had requested the helper extraction and `struct repository *` threading and reported that t5505 and t5586 fail when the series is in `seen`.

### doc: rewrite detached HEAD state explanation

*Long thread (root 2026/10/07/14-52-01): 20 replies so far from 3 participants over 14 days; 2 messages that day.*

Julia Evans continued defending her rewrite of the detached HEAD explanation as a standalone `gitdetached-head` guide wired into `advice.c` and `command-list.txt`. She confirmed that many potential readers do not understand HEAD and are doing fine, explained that she shared experienced users' own descriptions of HEAD to show the language users actually use, and said sorting through user comments is her job and needs some trust; she also added an XKCD reference to her earlier remark. Previously, Junio C Hamano had questioned what the quotes were meant to illustrate and recorded proposed replacement wording for the first paragraphs.

### doc: add new Git tutorial for beginners

*Long thread (root 2026/10/06/19-37-00): 19 replies so far from 5 participants over 15 days; 13 messages that day.*

Julia Evans's v2 series replacing `Documentation/gittutorial.adoc` with a new beginner tutorial drew review. Junio C Hamano called patch 2/2 well-paced and free of common beginner traps, with nitpicks including a missing hyphen in a Step 10 heading underline, and reported two AsciiDoc build warnings about section delimiters not preceded by an empty line. Julia Evans proposed switching all headings to Atx-style, noted ed25519 has been the OpenSSH default since 2023, and would upcase "Push your changes". Earlier today, the v2 patches had briefly been missing from the list archive; Ben Knoble suspected the list or archive had filtered them.

### odb: introduce object storage extension

*Long thread (root 2026/10/19/08-55-55): 14 replies so far from 3 participants over 2 days; 10 messages that day.*

Patrick Steinhardt's seven-patch series introducing the `extensions.objectStorage` extension -- configuring the primary object database source via a `<schema>[://<payload>]` URI, currently only the `files` schema -- drew close review. Patrick explained he removed the unused `repo_write_loose_object_map()` after realizing it would write to the wrong location under the extension, is retaining relative-path behavior for now, and traced the `object.format` versus `objects.storageFormat` naming clash to the earlier `references.format` naming; he agreed exposing the default format outside the repository and odb subsystems is awkward and will look at fixing it. Karthik Nayak accepted the reasoning on the test matrix and advised leaving any `object.format` rename out of this series.

### github-ci: update macOS jobs from macos-14

*Long thread (root 2026/10/16/22-46-36): 10 replies so far from 4 participants over 5 days; 8 messages that day.*

Junio C Hamano iterated on updating the four `osx-*` jobs in `.github/workflows/main.yml` after GitHub announced the macOS 14 runner image is closing down. He posted v3 using `macos-latest` for two jobs, dropped it in favor of v2, then posted v4 with explicit `macos-26` for two jobs and `macos-15` for the other two so github-ci and gitlab-ci can be compared on the same commit -- and then retracted v4 as bogus. Earlier, brian m. carlson, Todd Zullinger and Patrick Steinhardt had debated `macos-latest` versus explicit versions, with Patrick preferring explicit versions to keep both pipelines in sync.

## The day in brief

Harald Nordgren posted v10 of his series making shallow-repository fetches avoid fetching every branch of a new remote, and Patrick Steinhardt's object storage extension drew close review of its naming and tests. Julia Evans defended her detached HEAD guide and received nitpick-level review of her new beginner tutorial, while Junio C Hamano's macOS CI update was retracted after several revisions.

## Notable threads

### object-file: fix ".keep" lockfile ownership in ODB transactions

Justin Tobler posted v3 of his four-patch series fixing a bug where a `git-receive-pack` process could delete another concurrent process's `.keep` lockfile for an identical packfile, leaving the pack vulnerable to housekeeping. The series moves `.keep` lockfile generation to ODB transaction commit: `odb_transaction_files_write_pack()` now only records the pack name, and during commit a new `write_pack_lockfiles()` helper writes the lockfile via `write_pack_lockfile()` and registers it as a tempfile, so only the owning process can delete it. Patrick Steinhardt suggested leaving `ret` explicitly uninitialized so the compiler warns on unset paths; Justin Tobler said he doesn't mind sending another version correcting this.

### reftable/record: bound an object record's offset count by its length

Matthew Suozzo's patch bounds an object record's offset count by the record's remaining length, fixing an integer overflow in `reftable_obj_record_decode()` on 32-bit platforms. Patrick Steinhardt argued the `count > in.len` check misses the root cause -- `reftable_alloc_size()` taking `size_t` parameters that can wrap -- and proposed changing them to `uintmax_t`; he also recommended writing commit messages by hand rather than with AI. Aniket Gupta reported the fix works for `REFTABLE_ALLOC_ARRAY` on 32-bit builds but `REFTABLE_CALLOC_ARRAY` still truncates via `reftable_calloc()`.

### switch/checkout stops halfway when tracking config write fails

井上司 reported that `git switch`/`checkout` exits 1 mid-switch when writing the tracking config fails (e.g. `.git/config.lock` exists): the worktree and index are updated for the new branch but HEAD still points to the old one, traced to `exit(1)` in `setup_tracking()` in `branch.c`. Junio C Hamano preferred an atomic all-or-none outcome. 井上司 argued full atomicity would be a large change and proposed making `setup_tracking()` return an error instead of exiting; he later explained the order is worktree, ref, tracking config, then HEAD, with `merge_working_tree()` updating the worktree and index on every switch.

### worktree: add --reflink for copy-on-write worktree creation

Jason Newton posted v2 of his two-patch series adding `git worktree add --reflink`, which on copy-on-write filesystems reflinks the source worktree's files and index then runs `git reset --hard` so only differing paths are rewritten, plus ephemeral-branch reaping: a branch auto-created by `worktree add` is recorded in `worktrees/<id>/owned-branch` and deleted by `worktree remove`. brian m. carlson said the Windows ReFS block-cloning arm is defective due to a known Git LFS corruption bug (issue #6312) and should be skipped on Windows, and asked whether macOS `cp` supports `--reflink=always` so the APFS path gets CI coverage.

### fetch: commit references fetched before backfilling tags

Karthik Nayak's v3 patch fixes shallow protocol-v2 fetches where tag backfill could not advertise newly fetched refs in the `haves` section, making the server resend objects the client already has; it commits the batched ref updates before starting a new transaction for tag backfill. Justin Tobler confirmed the problem, noted it already existed for `--atomic` fetches which the patch does not fix, suggested supplementing the backfill `haves` with the pending transaction's references, and questioned why the code logs and continues when committing the previous transaction fails.

## In brief

- Harald Nordgren posted v2 of "upload-pack: fix slow fetches from a shallow repository", writing shallow lines through the packet writer instead of `deepen(data, INFINITE_DEPTH)`.
- Patrick Steinhardt posted a four-patch series restoring alternates reloading on missing objects, with optimizations bringing 20k missing-object lookups from 309ms to 197ms.
- Phillip Wood posted v3 of "stash creation: avoid unnecessary index loads and refreshes", cutting lstat() calls per changed path by four for `git stash create` and eight for `git stash push`.
- Matt Hunter posted a patch restoring the absolute path in `git init`'s "Initialized ... repository" message, a regression from 20e38cba3926. *Long thread (root 2026/10/20/05-38-47): 7 replies so far from 4 participants over 1 day; 8 messages that day.*
