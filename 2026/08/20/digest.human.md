# Git mailing list daily digest for 2026/08/20

## Long threads

### `git history squash` subcommand for folding commit ranges

*Long thread: 86 replies so far from 9 participants over 68 days; 2 messages that day.*

Harald Nordgren posted v14 of the eight-patch series adding `git history squash <revision-range>` to `builtin/history.c`, which folds a range of commits into one; v14 updates commit message trailers and simplifies the `rev-parse`/`tr` usage in the tests. Previously the implementation was split into five patches incorporating Phillip Wood's fixups, after Junio C Hamano had urged Harald to answer review comments in his own words before rerolling. Today Junio C Hamano questioned why patch 7/8 uses `istarts_with()` to match hex object IDs in `fixup!` subjects, noting that `fixup! ABCdef` should not be accepted.

### Pluggable packfile writes for `git-receive-pack`

*Long thread: 33 replies so far from 3 participants over 15 days; 5 messages that day.*

Justin Tobler posted v5 of the nine-patch series "builtin/receive-pack: support pluggable packfile writes", which introduces the generic `odb_transaction_write_pack()` interface so the existing child-process logic spawning `git-index-pack`/`git-unpack-objects` becomes the "files" backend implementation; v5 adds a test assertion that `.keep` files are migrated to the main ODB before removal at finalize. Patrick Steinhardt approved passing the unpack limit via `odb_transaction_flags` rather than assuming `git-receive-pack`, suggested extending the quarantine test to verify the full `.keep` lifecycle, and proposed a commit-message rewording; Justin agreed to add the lifecycle test in the next version.

### Eagerly loading ODB alternates

*Long thread: 29 replies so far from 5 participants over 11 days; 3 messages that day.*

Patrick Steinhardt's v3 series "odb: eagerly load alternates" initializes alternates up front instead of deferring `odb_prepare_alternates()` calls scattered across the codebase, adding a first patch that creates the ref and object databases after the repository config is written in `init_db()`. Today Karthik Nayak affirmed that patch's rationale -- `core.sharedRepository` must be known when the odb is created -- and said the new patch and range-diff look good. Junio C Hamano questioned the `eq()` semantics in patch 2/5 for object databases on filesystems with differing case sensitivity, objected that discarding the hash map does not resolve what `eq()` should be, and noted it is unclear whether `o->repo->initialized` is true in that code path.

### Pluggable packfile generation via the object database

*Long thread: 24 replies so far from 6 participants over 14 days; 14 messages that day.*

Patrick Steinhardt posted v3 of "odb: make packfile generation pluggable", converting `git-upload-pack`, `git-send-pack`, and `git-bundle` to generate packfiles through a new `odb_generate_pack()` interface with a "files" backend in `odb/source-files.c`; v3 fixes the use-after-scope bug Elijah Newren reported in `odb_source_files_generate_pack()`, where a stale pointer to a function-local `cp` could be dereferenced on abnormal exit. Karthik Nayak reviewed the patches, asking whether the unused `source` parameter is needed; Patrick explained it is kept for consistency for other backends and agreed the upload-pack error should say "unable to pack objects". Junio C Hamano asked whether `opts->pack_fd` should be closed when the primary source does not support `generate_pack`.

### Converting `USE_NSEC` to runtime config

*Long thread: 23 replies so far from 5 participants over 14 days; 5 messages that day.*

D. Ben Knoble posted v4 of "Convert USE_NSEC to runtime config" (topic `dk/use-nsec-runtime`), exposing `core.useNanosec` so index-related commands are less likely to hit racy-git problems on supported filesystems; v4 ignores the config variable only under `NO_NSEC` while using the `use_nanosec` member unconditionally, and mentions potential platform unsafety in the docs. Patrick Steinhardt welcomed removing the preprocessor directives in `is_racy_stat()` and agreed no warning should be emitted on `NO_NSEC` builds, since users sharing a `$HOME/.gitconfig` across machines would be annoyed. Junio C Hamano reviewed patch 2/3 aligning `repo_config_values_init` with the struct declaration, suggesting a wording tweak and calling it a "great improvement".

## The day in brief
Three series posted new revisions: Harald Nordgren's `git history squash` v14, Justin Tobler's pluggable receive-pack packfile writes v5, and Patrick Steinhardt's pluggable packfile generation v3 fixing a use-after-scope bug. D. Ben Knoble's runtime `core.useNanosec` v4 and Yoichi NAKAYAMA's worktree DWIM fix also advanced, with Junio C Hamano queuing the latter.

## Notable threads

### `worktree add` should not DWIM with `-b`/`-B`
Yoichi NAKAYAMA posted v4 fixing a regression from Git 2.42 where `git worktree add -b <new-branch> <path> <branch>` could still DWIM `<branch>` to a remote-tracking branch, ignoring `<new-branch>`; the patch adds an `else if (ac == 2 && new_branch)` branch in `builtin/worktree.c` plus a test that `git worktree add -b branch ../wt foo` fails with "fatal: invalid reference: foo". Junio C Hamano reviewed it, noting it is much smaller than the initial iteration, and said "Will queue."

### `git-history(1)` fixup breaking other worktrees
Toon reported that `git history fixup HEAD` in one worktree leaves another worktree showing staged changes it never made. Patrick Steinhardt agreed that `git-history(1)` should refuse updating branches currently checked out in another worktree and created GitLab work item 772 to schedule this for the next release cycle.

### Avoiding reftable stack reloads inside transactions
Karthik Nayak's series removes `REFTABLE_STACK_NEW_ADDITION_RELOAD` and avoids reloading the reftable stack when it is already locked within a reference transaction, cutting the `fstat()` calls that dominated ref creation. Jeff King reported a ~25% speedup for `update-ref --stdin` with 50,000 refs. Patrick Steinhardt noted libgit2 also uses the flag and suggested renaming `reftable_stack_new_addition()`; Karthik confirmed there is no longer a "don't reload" flow and agreed the libgit2 adaptation should be simple.

## In brief
- Kristoffer Haugsbakk posted v2 of "trailers: stop recognizing URLs as trailers", rebased on `master` now that the dependent topic has landed; the change fails trailer parsing when a separator is followed by `://` before any whitespace.
- Discussion of Artur Bieniek's `git pull --hard` proposal shifted toward Phillip Wood's `git pull --reset [--discard-changes | --autostash]` alternative, which Artur now prefers since it refuses to discard uncommitted changes by default.
- Patrick Steinhardt said Elijah Newren's series recovering when a multi-pack-index names a removed pack looks good overall, and asked whether a variant of the race affects callers not using `OBJECT_INFO_QUICK`.
- Patrick Steinhardt clarified the error-propagation logic in his `OBJECT_INFO_DIE_IF_CORRUPT` series: objects not found fail gracefully, found-but-corrupt die; Karthik Nayak proposed rewording a comment to "prefer other errors over not found errors".
- Yoichi NAKAYAMA posted v2 of the worktree repair fix, adding `read_gitfile_raw()` so `git worktree repair` can convert relative `.git` cross-references to absolute paths.
