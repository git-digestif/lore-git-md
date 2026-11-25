# Git mailing list daily digest for 2026/11/24

## Long threads

### fetch: limited-fetch and `--refmap` semantics for configured refspecs

*Long thread (root 2026/09/19/14-47-26): 88 replies so far from 7 participants over 67 days; 2 messages that day.*

This series changes `git fetch` and `git remote add` so a shallow single-branch clone can add a remote without downloading every branch: `remote.<name>.fetchRefmap` becomes the configured default for `--refmap`, `remote.<name>.fetchUpstreams` controls fetching local branches' upstreams, and `git remote add --limited-fetch` sets both. Earlier revisions settled that a refspec with its own non-empty destination ignores the refmap. Today Junio C Hamano reviewed v15 patch 1/7: he noted the commit message references a configuration added in 3/7 rather than 2/7, suggested tightening the documentation wording for destination-bearing refspecs, and confirmed that a command-line refspec with an explicit empty destination (`main:`) still maps through `--refmap`. In his review of patch 2/7 he raised open questions: with no configured `remote.<name>.fetch`, `git fetch --refmap=... <URL>` would fetch HEAD into FETCH_HEAD while silently ignoring the refmap; `--refmap=""` may leave tag auto-following enabled; and `git fetch --prune --refmap=...` still prunes based on `remote.<name>.fetch` rather than the refmap.

### branch: `--delete-integrated` for squash- and rebase-merged branches

*Long thread (root 2026/09/29/07-30-30): 74 replies so far from 7 participants over 57 days; 3 messages that day.*

This series adds `git branch --delete-integrated <pattern>`, which deletes branches whose changes landed upstream via squash or rebase merges and names the upstream commit; v10 made the file merge a plain text merge ignoring `.gitattributes` drivers and added `--dry-run` support for `-d`/`-D`. Earlier, Jasper Mayone verified v10 builds cleanly and passes t3200. Today Harald Nordgren agreed with extending `--dry-run` to branch create and rename but said this topic is probably the wrong place and asked Junio C Hamano's opinion; Junio replied that if dry-run is not introduced here the position holds, but if it is, users would expect unsupported operations to report that; Harald accepted, said he would avoid the word "yet" in such messages, and committed to adding dry-run to the remaining operations later.

### utf8: bounds fixes in `utf8_strnwidth()` and ANSI escape handling

*Long thread (root 2026/11/16/18-56-59): 36 replies so far from 3 participants over 9 days; 1 message that day.*

This series fixes `utf8_strnwidth()` in `utf8.c` disregarding its `len` argument, corrects its comment, and adds a counted variant of `display_mode_esc_sequence_len()` so escape sequences and glyph widths never read past the given boundary; v5 uses an array index in the counted variant to avoid `s + len` and O(n^2) degradation. Earlier Junio approved the core idea but required sign-off ordering, test, and comment fixes. Today Dirk Gouders said he would provide an example clarifying the externally observable symptom (an escape sequence reaching the end makes `utf8_width()` read past `len`), agreed to leave a comment that the new tests must be updated when the TODO at the bottom of `utf8_strnwidth()` is addressed, and reported that he abandoned an audit of what callers do with the returned widths because the ints often end up as width specifiers in format strings.

### http: exponential backoff, configurable retry status codes, `post_rpc()` retries

*Long thread (root 2026/10/31/15-57-00): 32 replies so far from 4 participants over 25 days; 5 messages that day.*

This series fixes the off-by-one retry count and Retry-After Trace2 timing in the HTTP 429 handling, adds exponential backoff capped by `http.maxRetryTime` when no Retry-After header is present, adds `http.retryStatusCodes`, restricts `post_rpc()` retries to read-only services so pushes are not retried, and defaults `http.retryAfter` to 10 seconds. Earlier Patrick Steinhardt objected to retrying non-idempotent POSTs and to making retryable codes the user's problem. Today Junio C Hamano reviewed v4 and found patches 1/5 and 2/5 "Looking good", called 3/5 "Overall looking very good" while suggesting the docs state that an empty `http.retryStatusCodes` disables all retries, confirmed the gzip header re-add and deferred error reporting in 4/5 while asking whether the `results` struct needs resetting before retry, and said he spotted nothing questionable in the cover letter.

### Coverity-reported overflow fixes across archive, http, wrappers, approxidate

*Long thread (root 2026/10/14/06-09-26): 20 replies so far from 6 participants over 42 days; 6 messages that day.*

This five-patch series addresses overflows in `archive-tar`'s `get_path_prefix()`, `http.c`'s `fwrite_sha1_file()` byte counter, `write_in_full()`/`read_in_full()`, and `approxidate()`; v2 dropped the GCC-only overflow macros and simplified the approxidate check. Today Junio C Hamano corrected that `read_in_full_or_error()` returns an int and that `read_in_full()` aborts with `BUG()` on oversized counts, identified `worktree.c:should_prune_worktree()` as the remaining convertible caller, confirmed the archive-tar guard (correcting pathlen to at least 101), and judged the http one-line `size_t posn` change good; Phillip Wood endorsed the `write_in_full()` change, listed caller follow-ups in `config.c`, `entry.c`, `parallel-checkout.c`, and `odb/streaming.c`, and approved the approxidate fix as good and much simpler; Junio called v2 a clear improvement over v1.

## The day in brief
Junio C Hamano gave positive review verdicts to the HTTP retry series and to the filter-branch and cvsexportcommit security fixes, while raising open questions about `--refmap` edge cases in the fetch series. Several series posted new revisions or drew detailed review: commit `--fixup` message options, submodule worktree configuration, and packfile lifecycle renames all saw active discussion.

## Notable threads

### filter-branch: validate hex oids in state-branch map
This patch makes `git filter-branch --state-branch` require non-empty lowercase hex object IDs in the `filter.map` blob before writing `../map/$from_commit`, closing a path-traversal hole where malformed keys could write outside the tempdir. Today Junio C Hamano confirmed the vulnerability (empty keys fail with "Is a directory", `../` keys escape the map directory), suggested explaining why spaces are allowed in the destination, and gave the verdict "Looks good. Thanks."

### cvsexportcommit: avoid shell in `-W` git calls
This patch chomps `$go_back_to` and switches the two `-W` `system()` calls in `git-cvsexportcommit.perl` to list form so branch names containing shell metacharacters like `;` are not interpreted by `/bin/sh`. Today Junio C Hamano confirmed Git refname rules permit such characters, called the list-form rewrite trivially correct and a great safety improvement, and gave a final "Looks good. Thanks."

### commit: allow `-m`/`-F`/`-c`/`-C` for all `--fixup` variants
Erik Cervin-Edin posted v4 of this series, which lets `-m`/`-F` supply the message body and `-c`/`-C` supply the message for every `--fixup` variant (plain, amend, reword) by refactoring `prepare_to_commit()` to write the `fixup!`/`amend!` subject before the message-source chain, removing the previous incompatibility checks.

### submodule: worktree-specific configuration
This series makes submodule settings worktree-specific under `extensions.worktreeConfig`, keeps `submodule.<name>.gitdir` relative for linked worktrees, and makes `git submodule init` copy an existing gitdir only when it belongs to the current worktree. Today Junio C Hamano reviewed v2: on 1/3 he asked for the status quo in the commit message and flagged a static-buffer hazard where `relative_path()` can return a pointer into `worktree_git_path()`'s ring buffer; on 2/3 he noted `is_submodule_active()` takes a path not a name and proposed `repo_config_set_multivar_in_file_gently()` for duplicate entries; on 3/3 he raised a symlink normalization mismatch in `submodule_git_dir_matches_current_worktree()` and suggested canonicalizing both sides.

### packfile: lifecycle function renames and global packfile list
Patrick Steinhardt's six-patch series renames packfile lifecycle functions (e.g. `add_packed_git()` to `packfile_from_idx()`), adds `packfile_free()`, inlines `open_packed_git_1()`, and introduces a global `all_packfiles` list for process-wide limits. Today Junio C Hamano corrected what `pack_report()` prints, questioned skipping `do_not_close` packs, flagged a possible `pack_open_fds` underflow when `fstat()` fails after `git_open()`, and welcomed the rename scope; brian m. carlson said they would like fast-import statistics disabled by default in a future series.

## In brief
- Kaartic Sivaraam proposed fixes for the EXPENSIVE test lock: re-verifying the lock owner's PID before removing a stale lock, setting the held flag before writing the PID file, and accepting the TOCTOU race as preferable to deadlock.
- Christian Couder said the Outreachy patch removing `the_repository` from `negotiator/default` looks good, and Junio C Hamano said he will mark it ready for `next`.
- Dirk Gouders posted v5 of the `gitcli` doc fixes restoring an accidentally reverted `--` wording; Junio C Hamano said it looked very good and would mark it ready for `next`.
- Dominik Maier posted v2 of the graph fix preventing left-skewed merge parents from fusing with the left column, adding an `idx < 0` guard and two new skewed-merge tests.
- Dominik Maier posted v2 deferring `--add-file` stat() until after option parsing so `--no-add-file` can cancel it, and fixing an out-of-bounds pointer advance in `write_zip_entry()` for extra/virtual files.
- Dominik Maier posted v2 of the config fix skipping backslash-continued lines when matching sections, handling CRLF, bare `\r`, and empty section headers.
- Wonseok Oh posted v2 making fast-import die on EOF after an inline `M` command instead of looping or writing an uninitialized OID.
- Phillip Wood called it a bug that `git history --update-refs=head` is not refused during a rebase in the same worktree; Aniket Gupta retracted his earlier claim and will make it refuse in both modes.
- Okhuomon Ajayi posted v2 of the notes repository-passing series limited to five preparatory patches; Junio C Hamano noted the "v2" label collides with an earlier series and suggested "v5". *Long thread (root 2026/10/31/10-45-26): 20 replies so far from 5 participants over 25 days; 6 messages that day.*
