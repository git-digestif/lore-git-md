# Git mailing list daily digest for 2026/08/17

## Long threads

### `git history squash` subcommand series

*Long thread: 85 replies so far from 9 participants over 65 days; 7 messages that day.*

This series adds `git history squash <revision-range>` to fold a range of commits into one, with v13 splitting the implementation into five patches covering parsing, range validation, branch protection, commit creation, and message editing. Today the discussion turned to attribution and a test detail: Phillip Wood noted that several patches are missing his `Signed-off-by` from his fixup patches and that it should come above Harald Nordgren's, and Kristoffer Haugsbakk advised the order `Signed-off-by: Phillip Wood`, `Helped-by: Phillip Wood`, `Signed-off-by: Harald Nordgren`, which Harald asked about. Separately, Junio C Hamano corrected his earlier framing, saying the `istarts_with()` use is new code introduced by this series with no reason to be case-insensitive, and that the `| tr` in the test must be removed for the topic to play nicely with `bc/restrict-hex-to-lowercase`.

### Moving libgit.a sources into a `lib/` directory

*Long thread: 38 replies so far from 12 participants over 124 days; 1 message that day.*

This RFC series moves roughly 700 libgit.a source files into a `lib/` directory to declutter the repository root and make entry points more discoverable. Today Junio C Hamano responded to Michael Montalbo's mechanical grouping experiment, observing that the earliest group (`connect.c`, `fetch-pack.c`, `remote-curl.c`, `send-pack.c`, `transport.c`, `upload-pack.c`) forms the "transport" group and matches intuition, while the other groups may not be clear-cut; he suggested carving conceptually clean groups into their own directories, as was done with `builtin/`, `refs/`, or `odb/`, without waiting to classify the remainder, and raised whether bisect and revision machinery could be separated.

### Pluggable packfile writes for `receive-pack`

*Long thread: 29 replies so far from 3 participants over 12 days; 1 message that day.*

This series introduces the generic `odb_transaction_write_pack()` interface so that `git-receive-pack`'s packfile writing becomes ODB-backend pluggable, with the existing child-process logic becoming the "files" backend implementation. Today Patrick Steinhardt agreed with Justin Tobler's earlier suggestion to start using transaction flags such as `ODB_TRANSACTION_RECEIVE` in the current series to differentiate callers of `odb_transaction_write_pack()`, noting that doing so makes it more obvious how future series will look.

### Eagerly loading ODB alternates

*Long thread: 26 replies so far from 5 participants over 8 days; 8 messages that day.*

This series makes alternates eagerly initialized when the object database is created and drops the `loaded_alternates` and `alternates_db` fields. Today Patrick Steinhardt revised his position, suggesting the case-sensitivity change be left out of this patch and documented as a NEEDSWORK area, since the refactoring simplifies the code but carries higher regression risk; Jeff King agreed that unconditional case-sensitive matching is his preference and flagged a timing hazard in caching `core.ignoreCase` before the repository is fully configured, suggesting `repo_ignore_case()` instead. Patrick acknowledged the gotcha and posted v3, which adds a first patch creating the object database after the complete repository configuration is written in `init_db()`.

### Packfile loading performance with many packs

*Long thread: 22 replies so far from 5 participants over 6 days; 2 messages that day.*

This patch adds a `skip_dup_check` parameter to `packfile_list_append()` to fix the O(N^2) regression when loading many packfiles. Today Patrick Steinhardt noted that `packfile_list_prepend()` has essentially the same duplicate-check problem, and Jeff King identified prepend calls in hot code paths, including the MRU adjustment from `find_pack_entry()`; he supplied a synthetic many-pack test showing `git rev-list --count foo` growing from 18 ms with 500 packs to 6351 ms with 16000 packs, and concluded that MRU updates should use constant-time movement within a doubly-linked list rather than an explicit duplicate check and removal.

## The day in brief

The `git history squash` series drew discussion of sign-off ordering and a case-sensitivity test detail, while the ODB alternates series advanced to v3 with repository configuration ordering addressed. Reviewers pushed the many-packfile performance thread toward constant-time MRU reordering, and Junio C Hamano said he would queue the use-after-free fix in object-name resolution.

## Notable threads

### Use-after-free fix in `get_oid_with_context_1()`
Shlok Kulshreshtha posted v3 of the fix that collapses two exits into one so `free(new_path)` happens after the last use of `cp`, adding a t1092 test for a relative path to a sparse directory. Junio C Hamano reviewed the patch as "nicely described", confirmed the fix matches the expected behavior, and said "Will queue."

### Warning on valueless `submodule.active`
Junio C Hamano questioned the proposed warn-and-continue behavior for a valueless `submodule.active`, arguing the existing hard error is sufficient for users to locate and fix the broken configuration and that without stopping the process, early error messages scroll away unnoticed; he concluded the suggested change is not sensible, and also flagged the `Signed-off-by` real-name requirement and overly long lines.

### `http.sslVerifyStatus` for stapled OCSP responses
Grayson Gordon submitted v4 of the patch adding `http.sslVerifyStatus` to check stapled OCSP responses, moving the four new tests into t5551 keyed on `$HTTPD_PROTO` so they run only in the https pass and changing the prerequisite to `test_might_fail`. Junio C Hamano reviewed v4, noted the code and documentation were unchanged and looked good, and said he will replace the patch.

## In brief

- Patrick Steinhardt posted v2 of "odb: make packfile generation pluggable", mostly removing `the_repository` dependencies in `bundle.c`; Junio C Hamano objected that an `extern` declaration for `get_log_output_encoding()` defeats the purpose of dropping `USE_THE_REPOSITORY_VARIABLE`.
- Kristoffer Haugsbakk posted v4 of the format-rev documentation fix, using a `[synopsis]` open block instead of a code block; Junio C Hamano said the series is hopefully now ready for `next`.
- On the worktree repair relative-path patch, Junio C Hamano criticized the commit message for hiding the details; Yoichi Nakayama explained the concrete absolute/relative mismatch and agreed to revise using a `read_raw_gitfile()`-like function.
- Lutz Lengemann posted v2 of the zsh completion fix for `git -C <path>`, now walking all words before the command and skipping global options and their arguments.
- Friel submitted a patch adding a `write_pack_file/wrote_bytes` Trace2 datum to `builtin/pack-objects.c` reporting total pack bytes written, including split packs.
