# Git mailing list daily digest for 2026/11/24

## Long threads

### fetch: limited-fetch and --refmap semantics for shallow clones

*Long thread: 88 replies so far from 7 participants over 67 days; 2 messages that day.*

Harald Nordgren's series adds `git remote add --limited-fetch` with `remote.<name>.fetchUpstreams`/`fetchRefmap` so a shallow single-branch clone does not download every branch of a new remote; earlier v15 made `--refmap` work without command-line refspecs and stopped refspecs with their own destination from being mapped. Today Junio C Hamano reviewed patches 1/7 and 2/7: on 1/7 he noted the commit message cites a config added in 3/7 and confirmed an empty-destination refspec still maps through `--refmap`; on 2/7 he questioned whether `git fetch --refmap=... <URL>` with no configured fetch should error, whether `--prune` should follow the refmap, and whether tag auto-following works when `--refmap=""` drops destinations.

### branch: --delete-integrated for squash- and rebase-merged branches

*Long thread: 74 replies so far from 7 participants over 57 days; 3 messages that day.*

Harald Nordgren's series adds `git branch --delete-integrated <pattern>` to delete branches whose changes landed upstream via squash or rebase merges; earlier v10 made the file merge ignore `.gitattributes` drivers and added `--dry-run` for `-d`/`-D`. Today the discussion turned to extending `--dry-run` to branch create/rename: Harald agreed with Junio C Hamano that this topic is probably the wrong place, Junio said users would then expect unsupported operations to report that, and Harald accepted, avoiding the word "yet" and planning to add dry-run to the remaining operations afterward.

### utf8: bounds fixes in utf8_strnwidth() and ANSI escape handling

*Long thread: 36 replies so far from 3 participants over 9 days; 1 message that day.*

Dirk Gouders' series fixes `utf8_strnwidth()` in `utf8.c` disregarding its `len` argument and corrects its comment, with counted variants of `display_mode_esc_sequence_len()` that never read past a buffer end; earlier v5 used an array index to avoid `s + len` and O(n^2) cost. Today Dirk said he would provide an example of the observable symptom of an escape sequence reaching the buffer end, and agreed to leave a comment that the new tests must be updated when the TODO at the bottom of `utf8_strnwidth()` is addressed.

### http: exponential backoff, configurable retry status codes, post_rpc() retries

*Long thread: 32 replies so far from 4 participants over 25 days; 5 messages that day.*

Ravi Mistry's series closes four gaps in the HTTP 429 retry support: an off-by-one retry count, no exponential backoff without `Retry-After`, no configurable retry status codes, and no retries in `post_rpc()`; earlier v4 limited `post_rpc()` retries to read-only services and honored an explicit `Retry-After: 0`. Today Junio C Hamano found patches 1/5 and 2/5 sound, suggested documenting that an empty `http.retryStatusCodes` disables all retries, confirmed the gzip header re-add in 4/5 while asking whether the `results` struct needs resetting before retry, and said he spotted nothing questionable after Patrick Steinhardt's earlier reviews.

### Coverity-reported overflow fixes across archive, http, and date code

*Long thread: 20 replies so far from 6 participants over 42 days; 6 messages that day.*

Johannes Schindelin's five-patch series fixes Coverity-reported overflows: a zero-length guard in archive-tar's `get_path_prefix()`, `size_t` for the byte position in http's `fwrite_sha1_file()`, `write_in_full()` returning int status, a new `read_in_full_or_error()`, and `approxidate()` rejecting quantities above `INT_MAX`; earlier v2 dropped GCC-only overflow macros. Today Junio C Hamano corrected that `read_in_full()` aborts with `BUG()` on oversized requests, confirmed an off-by-one in the archive-tar patch, and judged v2 a clear improvement; Phillip Wood endorsed the `write_in_full()` change and approved the simpler `approxidate` fix.

## The day in brief
Junio C Hamano gave positive verdicts to the HTTP retry series, the filter-branch state-branch validation, and the cvsexportcommit shell-safety fix, while Harald Nordgren's limited-fetch series drew detailed review with open `--refmap` questions and Dominik Maier posted a v2 fixing left-skewed merge rendering in `git log --graph`.

## Notable threads

### filter-branch: validate hex oids in state-branch map
Dominik Maier's patch makes `git filter-branch --state-branch` require non-empty lowercase-hex keys and hex-only values in a loaded `filter.map`, so a malformed state branch cannot write outside the temp map directory. Junio C Hamano confirmed the vulnerability (empty keys fail with "Is a directory", `../` keys escape the tempdir), suggested explaining why spaces are allowed in targets, and gave the verdict "Looks good. Thanks."

### cvsexportcommit: avoid shell in -W git calls
Dominik Maier's patch converts the two single-argument `system("git ... $go_back_to")` calls in `git-cvsexportcommit.perl`'s `-W` path to list-form `system('git', ...)`, since valid branch names can contain shell metacharacters like `;`. Junio C Hamano confirmed refname rules permit such characters, called the rewrite trivially correct, and gave a final "Looks good. Thanks."

### graph: do not fuse merge parents with the column to their left
Dominik Maier posted v2 of a `graph.c` fix for left-skewed merges, where `graph_insert_into_new_columns()`'s shortcut let a merge's second parent swallow the column to its left, losing the edge and, with four or more parents, reading before `graph->new_columns[0]`. The fix adds an `idx < 0` condition so the shortcut applies only to existing non-parent columns, plus 3- and 4-parent octopus tests in `t/t4215-log-skewed-merges.sh`.

## In brief
- Dominik Maier posted v2 deferring `--add-file` stat() until after option parsing and fixing an out-of-bounds pointer advance in `write_zip_entry()` when `--prefix` exceeds an extra file's path.
- **Junio on packfile fd underflow and renames**: Flagged possible pack_open_fds underflow when fstat() fails after git_open() in open_packed_git_1() inlining; welcomed add_packed_git() to packfile_from_idx() rename.

- Wonseok Oh posted v2 making fast-import die on EOF after an inline `M` command instead of writing an uninitialized object ID.
