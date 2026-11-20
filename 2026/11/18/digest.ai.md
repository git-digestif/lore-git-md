# Git mailing list daily digest for 2026/11/18

## Long threads

### branch: `--delete-integrated` for squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 59 replies so far from 7 participants over 51 days; 1 message that day.*

Harald Nordgren's series adds `--delete-integrated` to `git branch` so that branches squash- or rebase-merged on hosting services, whose tips are unreachable, are deleted when an upstream commit contains all their changes, naming that commit in the output. Earlier revisions refined index reading and added tests; v8 reworded commit messages. Today Junio C Hamano flagged a potential crash in patch 4/4: when a path originally a gitlink is replaced with different regular files on both the branch and upstream, `change_state_at()` returns `CHANGE_NEEDS_MERGE`, causing `merge_keeps_upstream()` to call `read_mmblob()` on the submodule's commit object and crash. He also noted `collect_branch_changes()` records the branch mode but not the base mode, and asked whether he was misreading the code flow.

### builtin/rerere: convert to subcommands

*Long thread (root 2026/10/27/10-46-43): 41 replies so far from 6 participants over 23 days; 4 messages that day.*

Patrick Steinhardt's 12-patch v4 series converts `git rerere` to proper subcommands via `OPT_SUBCOMMAND()`, with a documented backwards-incompatible change: options before the subcommand, such as `git rerere --skip-locked gc`, are no longer accepted, which Junio accepted. Today Patrick acknowledged that the `diff` documentation still wrongly claims extra arguments are passed to the system `diff` command and will fix it; he retracted his earlier claim that the new "too many arguments" error for `git rerere unknown` was an improvement, calling it a regression since the synopsis used to print, and will fix it; he also agreed `CALLOC_ARRAY(args_copy, args.nr)` may not allocate the terminating NULL. Junio refined the fix: `COPY_ARRAY()` can stay, but `CALLOC_ARRAY()` needs one more element.

### http: exponential backoff and retry configuration

*Long thread (root 2026/10/31/15-57-00): 25 replies so far from 4 participants over 19 days; 10 messages that day.*

Ravi Mistry's v3 series fixes the off-by-one retry count and Retry-After Trace2 timing in HTTP 429 handling, adds exponential backoff when no Retry-After header is present, adds `http.retryStatusCodes`, extends retries to `post_rpc()`, and defaults `http.retryAfter` to 10 seconds. Today Patrick Steinhardt reviewed all five patches: he objected to pushing retry configuration onto users in 3/5, and to retrying POST requests in 4/5 since they are not necessarily idempotent, for example `push_git()` to `git-receive-pack`. Ravi Mistry replied agreeing to restrict `post_rpc()` retries in v4 to read-only services (`git-upload-pack`, `git-upload-archive`) with a push test, to keep 429 as the default status code with 5xx opt-in, and to honor an explicit `Retry-After: 0` by inspecting the header when `CURLINFO_RETRY_AFTER <= 0`.

### repo: revision filtering for "repo structure"

*Long thread (root 2026/09/24/16-45-02): 20 replies so far from 5 participants over 56 days; 4 messages that day.*

Mark C. Chu-Carroll's patch adds `[<revs>...]` to `git repo structure` so the object set can be limited by rev expressions when diagnosing performance issues; specifying revs causes a double ref-walk because `setup_revisions()` does not count refs. Earlier Patrick Steinhardt approved v7, and Junio had noted the tests break under SHA-256. Today Mark posted v8 adding a SHA1 prerequisite to the new tests, planning to rework them later. Junio then raised concerns: the implementation would lose `--` without `PARSE_OPT_KEEP_DASHDASH`, `included_revision_args` should be based on `revs.rev_input_given` rather than `argc > 1`, and the error path leaks `revs` because `release_revisions()` is not called; he also noted hardcoded `.disk_size` values make the tests brittle across zlib implementations.

### utf8: counted variant of display_mode_esc_sequence_len()

*Long thread (root 2026/11/17/16-50-41): 13 replies so far from 3 participants over 2 days; 8 messages that day.*

Junio C Hamano's patch adds `display_mode_esc_sequence_len_counted(ptr, len)` so counted buffers in `utf8_strnwidth()`, `strbuf_utf8_replace()`, and `strbuf_add_wrapped_text()` never scan past their end, keeping the original NUL-terminated function as a wrapper. Today Dirk Gouders agreed to take over the patch as his own series. Andy Koppe noted the function also processes expanded custom formats and message text that may contain colon-separated SGR subparameters, and gave `\033\[[0-9:;]*m` as the complete SGR regex, confirming `\033[:;m` is valid; Dirk and Andy agreed not to mix the colon change into this patch.

## The day in brief
Patrick Steinhardt posted a four-patch series fixing the auto-maintenance reflog-expire condition that spawned `git reflog expire` on every run, with reftable optimizations cutting `git maintenance run --auto` from 853ms to 19ms. Junio C Hamano flagged a potential crash in the `--delete-integrated` branch-deletion patch when a gitlink is replaced by regular files on both sides. Ravi Mistry agreed to restrict HTTP POST retries to read-only services after Patrick Steinhardt objected that pushes are not idempotent.

## Notable threads

### refs/reftable: reflog expiration performance

*Long thread (root 2026/11/18/09-30-59): 9 replies so far from 4 participants over 1 day; 10 messages that day.*

Patrick Steinhardt posted a four-patch series after noticing auto-maintenance blocked ~800ms spawning `git reflog expire`: the `--auto` condition treated every reflog entry as unreachable, overcounting prunable entries. The series fixes the condition to evaluate only against `gc.reflogExpire`, skips writing a new reftable on dry-run or no-op expires, and fixes a documentation typo; benchmarks show `git maintenance run --auto` dropping from 853.2ms to 18.6ms. Karthik Nayak and Toon Claes reviewed with minor remarks; Junio asked whether the dry-run change is performance-only.

### fsmonitor: directory moved into working tree not watched
John Mellor reported that on Linux, moving a directory into the working tree from outside leaves the built-in fsmonitor daemon watching only its subdirectories, not the directory itself, so edits to files directly inside it are invisible to status and related commands. He diagnosed `rename_dir()` in `compat/fsmonitor/fsm-listen-linux.c` not calling `add_watch(path)` in the "moved from outside" branch, and suggested calling `add_watch(path, data)` there.

### apply: reject corrupt "diff --git" headers
Tanga Joris's Outreachy series makes `git apply --check` report corrupt `diff --git` headers as errors instead of dying on an assertion. Today Junio C Hamano argued the 1/3 test only passes because the bogus line names the same file as the next line, and that such a line should be diagnosed as corrupt input. Tanga agreed `find_header()` should stop and report a corrupt patch when a `diff --git` line has nothing after it, making 1/3 unnecessary, and asked whether that matched Junio's intent before sending v4.

### diff-lib: skip leading path check for fsmonitor-valid entries
Maik Riechert posted a patch making `check_removed()` return immediately for entries with `CE_FSMONITOR_VALID`, skipping the `has_symlink_leading_path()` lstat walk; fsmonitor would have reported any leading-directory symlink replacement and cleared the bit. `git diff HEAD` on 100,000 files drops from 57ms to 26ms. Junio C Hamano reviewed it as OK assuming the valid bit is trustworthy, with a style suggestion.

### object-file: loose object size checks
Elia Pinto's v2 series widens object sizes to `size_t` on LLP64 platforms. Today Patrick Steinhardt agreed `unpack_loose_rest()` should verify the inflated stream produces exactly the header-declared size and report mismatches as corrupt, suggested replacing `strlen()` with `memchr()`, and asked whether the error should name differing expected and actual sizes. Junio clarified the "opposite case" where initial inflation already covers header and payload.

## In brief
- Adrian Ratiu posted a three-patch series fixing recursive-checkout failures when a submodule is a linked worktree, replacing a reachable `BUG()` with an error; Junio questioned the `is_worktree` check placement for nested submodules.
- Christian Couder reviewed the `repo.concern.*` threshold configuration patch with type-safety suggestions such as `size_t` and `git_config_size_t()`.
- Patrick Steinhardt reviewed Elia Pinto's pack size_t series, criticizing verbose commit messages and questioning casts pending another in-flight series.
- Junio C Hamano questioned the reworded missing-upstream status message's `push_creates()` logic, asking about narrow push refspecs and `push.default=nothing`.
- Pushkar Singh posted v4 making manual maintenance fail on lock acquisition while automatic and scheduled runs skip non-fatally, with Trace2 `lock/skip` events.
- Junio C Hamano approved the tree-walk size_t series, suggesting a constant sanity check be hoisted to `init_tree_desc()`.
- Noam Cohen posted a patch making `git ls-tree -z --format` output `%(path)` verbatim; Junio called the change trivially correct.
- Dirk Gouders posted v2 fixing `utf8_strnwidth()` bounds and its comment; Andy Koppe noted the comment also wrongly says "null-terminated".
- Christian Couder's series widens `odb_write_object()` to `size_t`; Patrick Steinhardt noted overlap with Elia Pinto's series and Couder will rebase.
- Julia Evans reported test-reader feedback on her branches tutorial and suggested a table of contents and a future remotes guide.
- Jakub Kulik posted a fix quoting redirection targets in t1017; Junio will queue it.
- Jakub Kulik posted a fix accepting positive `uname(2)` returns; Junio approved.
