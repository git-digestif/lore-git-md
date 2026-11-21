## Long threads

### `git history squash` subcommand series
*Long thread: 86 replies so far from 9 participants over 68 days.*
Harald Nordgren posted v14 of the eight-patch series adding `git history squash <revision-range>` to `builtin/history.c`, updating commit message trailers and simplifying `rev-parse`/`tr` usage in tests. Earlier in the week, Phillip Wood noted missing `Signed-off-by` trailers from his fixup patches and Kristoffer Haugsbakk advised a specific trailer ordering, while Junio C Hamano corrected his earlier framing on `istarts_with()` being new code that should not be case-insensitive, and later questioned why patch 7/8 uses `istarts_with()` to match hex object IDs in `fixup!` subjects. The series remains under active review with attribution and case-sensitivity details being resolved.

### Moving libgit.a sources into a `lib/` directory
*Long thread: 38 replies so far from 12 participants over 124 days.*
Junio C Hamano responded to Michael Montalbo's mechanical grouping experiment for this RFC series that moves roughly 700 libgit.a source files into a `lib/` directory, observing that the earliest group (`connect.c`, `fetch-pack.c`, `remote-curl.c`, `send-pack.c`, `transport.c`, `upload-pack.c`) forms a coherent "transport" group matching intuition, while other groups may not be clear-cut. He suggested carving conceptually clean groups into their own directories, as was done with `builtin/`, `refs/`, or `odb/`, without waiting to classify the remainder, and raised whether bisect and revision machinery could be separated.

### Pluggable packfile writes for `git-receive-pack`
*Long thread: 35 replies so far from 3 participants over 16 days.*
Justin Tobler posted v5 of the nine-patch series introducing the generic `odb_transaction_write_pack()` interface so `git-receive-pack(1)` writes incoming packfiles through the ODB transaction layer rather than child processes tied to the "files" backend. Patrick Steinhardt approved passing the unpack limit via `odb_transaction_flags` and suggested extending the quarantine test to verify the full `.keep` lifecycle, while Junio C Hamano called reading `receive.unpackLimit` and `transfer.unpackLimit` in generic object-layer code a layering violation and suggested `odb-file.unpackLimit` as a more accurate name. The series is nearing readiness with Patrick Steinhardt expressing happiness with v5.

### Eagerly loading ODB alternates
*Long thread: 31 replies so far from 5 participants over 12 days.*
Patrick Steinhardt's v3 series initializes alternates up front instead of deferring `odb_prepare_alternates()` calls, adding a first patch that creates the ref and object databases after the repository config is written in `init_db()`. Karthik Nayak affirmed the rationale that `core.sharedRepository` must be known when the odb is created, while Junio C Hamano questioned the `eq()` semantics for object databases on filesystems with differing case sensitivity and noted it is unclear whether `o->repo->initialized` is true in that code path. Patrick argued the alternate-deduplication mechanism is imperfect but has worked for roughly 20 years, and Junio proposed a best-effort case-insensitive match where the platform hints at one, concluding the code can stay as-is.

### Pluggable packfile generation via the object database
*Long thread: 31 replies so far from 6 participants over 15 days.*
Patrick Steinhardt posted v4 of the series converting `git-upload-pack`, `git-send-pack`, and `git-bundle` to generate packfiles through a new `odb_generate_pack()` interface with a "files" backend, fixing the use-after-scope bug Elijah Newren reported in `odb_source_files_generate_pack()` where a stale pointer to a function-local `cp` could be dereferenced on abnormal exit. Karthik Nayak reviewed v4 and said everything looks good, while Elijah Newren proposed an alternative for the `the_repository` cleanup in `bundle.c`, which Patrick defended. Junio C Hamano asked whether `opts->pack_fd` should be closed when the primary source does not support `generate_pack`, and Patrick confirmed it is consumed by the generator.

### Ambiguous remote branch message in `worktree add`
*Long thread: 27 replies so far from 6 participants over 15 days.*
Yoichi Nakayama posted v7 of the three-patch series fixing the unhelpful error when `git worktree add` or `git checkout` matches multiple remote tracking branches with the same name, rewording the advice to "Branch name '%s' appears in multiple remotes:" followed by the matched remote names. Junio C Hamano reviewed v7, called both changes sensible, said he very much likes the new, more concise advice message, and said "Will queue." He also classified the multiple-match-but-success case in `git worktree add` as the same class of bug, fine to leave for a later follow-up, noting that creating the branch from HEAD on multiple matches is buggy and should eventually error out.

### New path keys for `git repo info`
*Long thread: 25 replies so far from 4 participants over 37 days.*
K Jayatheerth's GSoC series adds `path.*` keys to `git repo info` for repository component paths -- toplevel, superproject root, hooks, index, grafts, git-prefix -- each with absolute and relative suffixes. He explained why the `path.toplevel` test does not use the `test_repo_info_path` helper, and agreed with Lucas Seiki Oshiro that `path.git-prefix` is technically current-directory information but kept it as useful for scripts. Junio C Hamano countered that `path.git-prefix` is repository information expressed as a relative path -- the inverse of a `path.cdup` -- and K Jayatheerth committed to adding a `cdup` key in the next version.

### diff hunk provider interface rethought
*Long thread: 25 replies so far from 4 participants over 94 days.*
Michael Montalbo retracted the "diff provider" abstraction introduced in RFC v7, saying it shoehorns several concepts together that should be addressed separately, and clarified that "chain order is authority" was an unclear way to describe earlier providers overriding xdiff's default. He confirmed the original intent was to configure a provider for a file type via `.gitattributes` and `diff.<driver>.process`, using xdiff when no provider is configured, and stated his goals: a "cousin" interface to xdiff-interface for user-configured alternatives, OID-only operation for callers that need only hunk header info, and a consistent pkt-line interface for plugging in xdiff alternatives. He decided to return to the v6 line for future re-rolls.

## The period in brief
The week of 2026/08/17 through 2026/08/23 saw seven active days of traffic dominated by long-running ODB abstraction work and several series reaching queue-ready status. Readers should not miss Elijah Newren's multi-pack-index race fix, Patrick Steinhardt's parallel packfile URI fetch series with benchmarked speedups, and the controversy over an AI-generated treewide `utime.h` migration that brian m. carlsson declined to review over DCO concerns.

## Key developments

### Multi-pack-index recovery after geometric repack
Elijah Newren posted a two-patch series addressing a race where geometric repacking creates a new multi-pack-index and removes the pack an older multi-pack-index named as owner, making present objects look missing. The series makes `git replay` fail gracefully with an error message instead of SIGSEGV when a merge input is unreadable, and teaches `find_pack_entry()` to scan the multi-pack-index's covered packs directly for a surviving copy. Junio C Hamano confirmed the diagnosis on the replay fix -- any value other than `0 < result->clean` is a failure, but the code mistakenly treated any non-zero value as OK -- and said he will prepare an evil-merge to adapt a `packfile_fill_entry()` call to an API change from another in-flight topic. Patrick Steinhardt later asked whether a variant of the race affects callers not using `OBJECT_INFO_QUICK`, and Elijah identified that it also affects `OBJECT_INFO_QUICK` callers like upload-pack's existence checks.

### Generic `OBJECT_INFO_DIE_IF_CORRUPT` handling in the ODB
Patrick Steinhardt posted v2 of a seven-patch series restructuring corrupt-object handling around `enum odb_read_status` with negative error codes (`ODB_READ_OK`, `ODB_READ_ERROR`, `ODB_READ_NOT_FOUND`), adding an `errmsg` parameter to `read_object_info()` so sources can bubble up human-readable corruption messages, and making the die-if-corrupt check backend-agnostic instead of relying on `has_packed_and_bad()`. Junio C Hamano accepted the explanation but noted the reasoning relies on the invariant that a single pack contains each object at most once, and Patrick clarified that objects not found fail gracefully while found-but-corrupt die.

### Parallel packfile URI fetches in `fetch-pack`
Patrick Steinhardt submitted a two-patch series letting `git-fetch(1)` and `git-clone(1)` fetch packfile URIs in parallel, refactoring `fetch-pack.c` into preparation, fetch and aggregation phases and adding the `fetch.packfileURIThreads` config (default 1, 0 for auto-detect), with benchmarks showing speedups up to 7.58x on a throttled connection.

### Treewide `utime.h` migration declined over AI provenance
A series replacing the POSIX-removed `utime()` and `<utime.h>` with `utimensat()` drew a refusal to review from brian m. carlsson because its cover letter disclosed it was generated by Gemini Flash from Antigravity, arguing this cannot satisfy the Developer's Certificate of Origin requirement. Junio C Hamano endorsed that stance as prudent, explaining that the contributor's DCO cannot be treated as valid because a copy editor of AI output cannot know where the code was copied from, and that the project itself bears the cost if DCO-covered work later proves legally unusable.

### `git organize` RFC records and applies file placement
Michael Montalbo posted RFC PATCH 00/14 introducing `git organize`, a command that records a project's file-placement guideline in a tracked `.gitorganize` file and moves out-of-place files, repointing references and staging the result. It has `status` and `apply` subcommands, configurable `organize.labeler`/`organize.organizer` commands, and eight planned carves for odb, refs, pack, diff, revision, index, setup, and transport; the final patch moves 22 transport C sources and 21 headers into `transport/`.

### `git config --global` to read home and XDG files
Delilah Ashley Wu posted v2 of a three-patch series making `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`, fixing the inconsistency where `--global` read only the former while plain `git config list` read both. The patches enforce forward slashes in XDG paths on Windows, let the config sequence optionally report an error when no files were read, and make `--global` reuse `do_git_config_sequence()`. Chris Torek asked which of the two files `git config --global --edit` edits.

### Use-after-free fix in `get_oid_with_context_1()` queued
Shlok Kulshreshtha posted v3 of the fix that collapses two exits into one so `free(new_path)` happens after the last use of `cp`, adding a t1092 test for a relative path to a sparse directory. Junio C Hamano reviewed the patch as "nicely described", confirmed the fix matches the expected behavior, and said "Will queue."

### `worktree add` should not DWIM with `-b`/`-B` queued
Yoichi NAKAYAMA posted v4 fixing a regression from Git 2.42 where `git worktree add -b <new-branch> <path> <branch>` could still DWIM `<branch>` to a remote-tracking branch, ignoring `<new-branch>`; the patch adds an `else if (ac == 2 && new_branch)` branch in `builtin/worktree.c` plus a test. Junio C Hamano reviewed it, noting it is much smaller than the initial iteration, and said "Will queue."

## In brief

- **Stop recognizing URLs as trailers** -- Kristoffer Haugsbakk posted v3 of the patch that makes trailer parsing fail on `://` separators so HTTPS URLs at the end of commit messages are not mistaken for trailers; Junio C Hamano marked the topic for `next`.
- **Avoid sending the whole tree when pushing from a shallow clone** -- Elijah Newren submitted a `send-pack` patch adding reachable shallow grafts as negative pack tips controlled by `push.shallowExcludeBoundary` (default true); Patrick Steinhardt reviewed it with questions about graft amendment and tree negotiation.
- **`http.sslVerifyStatus` for stapled OCSP responses** -- Grayson Gordon posted v6 with documentation rephrased per Junio C Hamano's suggestion; Junio said he will replace the patch.
- **Incoming-proportional connectivity check** -- Kristofer Karlsson proposed an RFC to make `check_connected()` cost proportional to the incoming set rather than repository size, reporting prototype results dropping the check from about 1s to about 0.03s for a five-commit push on a repository with 3M commits and 200K refs.
- **Avoiding reftable stack reloads inside transactions** -- Karthik Nayak's series removes `REFTABLE_STACK_NEW_ADDITION_RELOAD` and avoids reloading the reftable stack when already locked, with Jeff King reporting a ~25% speedup for `update-ref --stdin` with 50,000 refs.
- **Report hook for `git-receive-pack(1)`** -- Karthik Nayak posted v2; Junio C Hamano agreed a failed report hook should explicitly tell the client the push was declined, modeled on pre-receive.
- **Converting `USE_NSEC` to runtime config** -- D. Ben Knoble posted v4 exposing `core.useNanosec`; Junio C Hamano called patch 2/3 a "great improvement" and agreed no warning should be emitted on `NO_NSEC` builds.
- **Packfile loading performance with many packs** -- Jeff King supplied a synthetic many-pack test showing `git rev-list --count foo` growing from 18 ms with 500 packs to 6351 ms with 16000 packs, concluding MRU updates should use constant-time movement within a doubly-linked list.
- **`git-history(1)` fixup breaking other worktrees** -- Toon reported that `git history fixup HEAD` in one worktree leaves another showing staged changes; Patrick Steinhardt agreed it should refuse updating branches checked out in another worktree and created GitLab work item 772.
- **credential/libsecret: load secrets explicitly** -- Daniel Martí defended his patch dropping `SECRET_SEARCH_LOAD_SECRETS` in favor of explicit `secret_item_load_secret_sync()`, arguing the flag discards errors from the second step and skips locked items client-side.
- **Bug Report: subtree then rebase causes unrelated history to conflict** -- Gabriel Ford reported that after `git subtree add --squash`, a later `git rebase` conflicts between files with the same relative path in separate directories.
- **configurable ref for the stash stack** -- Vladimir Sitnikov proposed a `stash.ref` configuration knob (defaulting to `refs/stash`) so a worktree can use an isolated stash stack; Kristoffer Haugsbakk agreed a configuration option makes the most sense and answered that no ref namespace is special for GC.
- **zsh completion fix for `git -C <path>`** -- Lutz Lengemann posted v2 walking all words before the command and skipping global options and their arguments.
- **`pack-objects` trace patch** -- Friel's patch adding a `wrote_bytes` Trace2 datum got "looks good" verdicts from Jeff King and Junio C Hamano.
- **`git pull --hard` vs `git pull --reset`** -- Discussion of Artur Bieniek's proposal shifted toward Phillip Wood's `git pull --reset [--discard-changes | --autostash]` alternative, which Artur now prefers since it refuses to discard uncommitted changes by default.
- **worktree repair relative-path fix** -- Yoichi NAKAYAMA posted v3 adopting a caller-prepared `strbuf` signature for `read_gitfile_raw()` so `git worktree repair` can convert relative `.git` cross-references to absolute paths.
- **Volodymyr Vriukalo fixed `git branch --recurse-submodules`** -- guarding the tracking-name call when the start point names no ref and making the helper's start-name argument optional.
- **Oswald Buddenhagen proposed an opt-out for Unicode box-drawing output** -- in `git repo structure`; Junio C Hamano agreed an opt-out is needed since locale-only detection leaves no way to disable it.
- **Johannes Sixt submitted a gitk README patch discouraging AI contributions** -- Weijie Yuan asked how "Generated" differs from "Assisted" content, and Weijie Yuan and Junio C Hamano discussed AI-assistance disclosure in patches.
- **Kristoffer Haugsbakk posted v2 advertising gitdatamodel(7)** -- across documentation, correcting commit messages.
- **Nikolaus Schuetz clarified refname character rules** -- that dot is the only refname character with position-specific rules, so the t1402 patch only needed to test other forbidden characters embedded in a refname.