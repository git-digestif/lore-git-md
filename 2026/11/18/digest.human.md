# Git mailing list daily digest for 2026/11/18

## Long threads

### branch: `--delete-integrated` for squash-merged branches

*Long thread: 59 replies so far from 7 participants over 51 days; 1 message that day.*

Harald Nordgren's series adds `--delete-integrated` to `git branch`, deleting branches whose changes landed in an upstream commit even when the tip is unreachable, as with GitHub squash or rebase merges, and naming that commit in the output. Today Junio C Hamano flagged a potential crash: when a gitlink path is replaced with different regular files on both the branch and upstream, `change_state_at()` returns `CHANGE_NEEDS_MERGE`, making `merge_keeps_upstream()` call `read_mmblob()` on the submodule's commit object and crash; he also noted `collect_branch_changes()` records the branch mode but not the base mode.

### builtin/rerere: convert to subcommands

*Long thread: 41 replies so far from 6 participants over 23 days; 4 messages that day.*

Patrick Steinhardt's series converts `git rerere` to `OPT_SUBCOMMAND()` parsing, including a settled backwards-incompatible change requiring options after the subcommand. Today Patrick acknowledged the `diff` documentation still wrongly claims extra arguments go to the system `diff` and will fix it; retracted his claim that the new "too many arguments" error for `git rerere unknown` was an improvement, calling it a regression since the synopsis used to print; and agreed `CALLOC_ARRAY(args_copy, args.nr)` may miss the terminating NULL, which Junio refined: `COPY_ARRAY()` can stay but `CALLOC_ARRAY()` needs one more element.

### http: retry backoff, status codes, and `post_rpc()` retries

*Long thread: 25 replies so far from 4 participants over 19 days; 10 messages that day.*

Ravi Mistry's v3 series fixes HTTP 429 handling in the http transport: an off-by-one retry count, Retry-After Trace2 timing, exponential backoff without a Retry-After header, a configurable `http.retryStatusCodes`, retries in `post_rpc()`, and a 10-second default for `http.retryAfter`. Today Patrick Steinhardt objected that retrying `post_rpc()` is unsafe for non-idempotent POSTs like `git-receive-pack` pushes; Ravi agreed to restrict v4 retries to read-only services, honor an explicit "Retry-After: 0", and make `attempt` a `size_t`.

### repo: revision filtering for "git repo structure"

*Long thread: 20 replies so far from 5 participants over 56 days; 4 messages that day.*

Mark C. Chu-Carroll's patch lets `git repo structure` take rev expressions limiting which objects are processed, to diagnose performance issues in a specific part of a repository. Today he posted v8, adding a SHA1 prerequisite to the new tests after Junio reported they break under SHA-256; Junio then raised concerns that the implementation drops "--" without `PARSE_OPT_KEEP_DASHDASH`, that `included_revision_args` should key off `revs.rev_input_given` rather than `argc > 1`, and that the error path leaks `revs` without calling `release_revisions()`.

### utf8: counted ANSI escape sequence length

*Long thread: 13 replies so far from 3 participants over 2 days; 8 messages that day.*

Junio C Hamano's patch adds `display_mode_esc_sequence_len_counted(ptr, len)` so counted buffers in `utf8_strnwidth()`, `strbuf_utf8_replace()` and `strbuf_add_wrapped_text()` never read past their end, keeping the original function as a NUL-terminated wrapper. Today Junio asked Dirk Gouders to take over the patch; Andy Koppe noted expanded custom formats can contain colon-separated SGR sequences whose width is not currently ignored, gave `\033\[[0-9:;]*m` as the complete SGR regex, and confirmed `\033[:;m` is valid; Dirk acknowledged and will read ECMA-48.

## The day in brief
The day's most consequential exchanges were Junio C Hamano's crash report on the `--delete-integrated` branch patch, Patrick Steinhardt's objection to retrying non-idempotent HTTP POSTs in the http retry series, and Patrick's reflog-expiration series cutting auto-maintenance from about 853ms to under 19ms.

## Notable threads

### refs/reftable: reflog expiration performance

*Long thread: 9 replies so far from 4 participants over 1 day; 10 messages that day.*

Patrick Steinhardt posted a four-patch series fixing the auto-maintenance "reflog-expire" condition, which treated every reflog entry as unreachable and spawned `git reflog expire` on every run, blocking the foreground for ~800ms; the series fixes the condition and skips writing new reftables on dry-run or no-op expires, with `git maintenance run --auto` dropping from 853.2ms to 18.6ms. Karthik Nayak, Toon Claes and Junio C Hamano reviewed, with Junio asking whether the dry-run change is performance-only.

### apply: reject corrupt "diff --git" headers
Tanga Joris's series makes `git apply --check` report corrupt "diff --git" headers as errors instead of dying on an assertion. Today Junio C Hamano argued the 1/3 test only passes because the bogus line names the same file as the next line; Tanga confirmed a different name yields "inconsistent old filename", agreed `find_header()` should report such a line as corrupt patch input, and asked whether that matched Junio's intent before sending v4.

### submodule: linked worktrees in recursive checkout
Adrian Ratiu posted a three-patch series fixing recursive-checkout failures, a BUG() crash or a broken submodule, when a submodule is a linked worktree of its repository: replacing the BUG() with an error, gating the `submodulePathConfig` advice on the extension being disabled, and accepting a linked-worktree git dir. Junio C Hamano reviewed, suggesting comment rewording and questioning whether a nested-submodule case could still reach the force-reconnect block with `is_worktree == 0`.

## In brief
- John Mellor reported that on Linux the fsmonitor daemon does not watch a directory moved into the working tree from outside, only its subdirectories, so edits to files directly inside it are invisible to status; he traced it to `rename_dir()` in `compat/fsmonitor/fsm-listen-linux.c` not calling `add_watch(path)` in the moved-from-outside branch.
- Maik Riechert posted a diff-lib.c patch making `check_removed()` return immediately for `CE_FSMONITOR_VALID` entries, skipping the `has_symlink_leading_path()` lstat walk; `git diff HEAD` on 100,000 files drops from 57ms to 26ms, and Junio C Hamano called it OK assuming the valid bit is trustworthy.
- Dirk Gouders posted v2 of utf8 bounds fixes using `display_mode_esc_sequence_len_counted()`; Junio praised patch 4/4, and Andy Koppe noted the `utf8_strnwidth()` comment also wrongly calls the input null-terminated.
