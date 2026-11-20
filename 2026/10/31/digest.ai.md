# Git mailing list daily digest for 2026/10/31

## Long threads

### branch: --delete-integrated for squash- and rebase-merged branches

*Long thread (root 2026/09/29/07-30-30): 53 replies so far from 6 participants over 33 days; 2 messages that day.*

Harald Nordgren's series adds a `--delete-integrated <pattern>` option to `git branch` that deletes branches whose changes landed upstream through squash or rebase merges, even when the branch tip is not reachable from the upstream, naming the upstream commit in output like "Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)". The latest revision keeps `--delete-merged` limited to branches whose tips are reachable and makes the two options mutually exclusive. Today Harald Nordgren suggested renaming the local variable `opt` to `diffopt` in `find_landed_commits()` so the name matches the `diff_options` type, and he questioned whether `merge_keeps_upstream()` ensures the index is read before calling `ll_merge()` with `the_repository->index`, since an unpopulated istate would prevent `ll_merge()` from consulting custom low-level merge attributes; he asked whether ignoring attributes is intentional and should be stated explicitly, and said he would take a look.

## The day in brief

Linus Torvalds submitted a fix for `git describe` miscomputing commit distance when commit dates are skewed, and Nakul V S posted v2 of a series refusing to pick or revert shallow boundary commits. Ravi Mistry posted a four-patch series hardening HTTP 429 retry handling, while Harald Nordgren continued refining `--delete-integrated` and `@{push}` resolution for triangular workflows.

## Notable threads

### describe: count distance correctly with skewed commit dates

Linus Torvalds submitted a patch fixing `git describe` in `builtin/describe.c` miscomputing the commit distance when commit dates are skewed, as seen in the Linux kernel where `git describe` reported v7.3-19342-g2a500a04d6f9 while the kernel's own calculation gave 7.3.0-19059-g2a500a04d6f9. The fix changes the priority queue ordering from `compare_commits_by_commit_date` to `compare_commits_by_gen_then_commit_date` and adds a test in `t/t6120-describe.sh` for a tag dated before its parent. Tuomas Ahola shared an alternative rewrite of the test case using `test_tick` arithmetic instead of explicit dates.

### sequencer: refuse shallow boundary commits

Nakul V S posted v2 of a patch making the sequencer refuse to pick or revert shallow boundary commits by default across revert, cherry-pick, and rebase (merge, interactive, and --apply backends), since reverting such a commit removes every tracked file. `--allow-shallow-boundary` overrides the check for revert and cherry-pick only and is remembered across `--continue`, while rebase always refuses and suggests `git fetch --unshallow`. The check now runs where each command builds its list of commits, so a refused command leaves no sequencer or rebase state behind; `git commit --amend` is left for a separate patch.

### http: exponential backoff and post_rpc() retries

Ravi Mistry posted a four-patch series addressing gaps in HTTP 429 retry handling in `http.c` and `remote-curl.c`: fixing an off-by-one retry count where `http.maxRetries=N` performed only N-1 retries and moving Retry-After extraction so the Trace2 event logs the actual value; adding exponential backoff when no Retry-After header is present; adding an `http.retryStatusCodes` config option defaulting to "429" so environments can opt into retrying codes such as 502, 503, and 504; and extending retries to HTTP POST requests in `probe_rpc()` and `post_rpc()`.

### parse-options: early_scan_options() for fast-import

Christian Couder's series introduces an `early_scan_options()` sub-API in the parse-options API so commands like `fast-import` can safely skip option values during early argument scans. Cynthia Uche reported that all 118 t0040 tests and all 261 t9300 tests pass on the topic, and examined patch 4/4's use of `EARLY_SCAN_STOP_AT_NON_OPTION`, demonstrating that with the flag `git fast-import --depth 5 extra --allow-unsafe-features` dies with "forbidden in input without --allow-unsafe-features" after the stream's feature lines, while without it the scan passes "extra" and only fails later with a usage error; she suggested a comment sentence explaining the flag and found no other issues.

### remote: resolve "simple" @{push} for another push remote

Harald Nordgren posted v2 of a series making `@{push}` resolve to where `git push` goes when `push.default=simple` and `remote.pushDefault` points at a fork, where it previously failed with "cannot resolve 'simple' push to a single destination". The v2 moves the "simple" resolution out of `branch_get_push_1()` into a helper `branch_get_push_simple()` and makes `@{push}` resolve to the same-named branch on the push remote in triangular workflows, with tests updated for the error path.

## In brief

- Jeff King fixed t5702 to check `GIT_PROTOCOL` via direct `echo` output compared with `test_cmp` instead of grepping `env` output, which GNU coreutils 9.12 now quotes; Junio C Hamano confirmed the patch is correct.
- brian m. carlsson objected to ignoring include directives in `.git/config`, saying his team relies on them and Git should not break that behavior.
- Jasmeet (Jazz) Bhatia posted a patch reusing the prepared untracked file list in `do_create_stash()`, reporting stash -p gains of 16.7% without fsmonitor and about 6% on Linux kernel testing.
- Thomas Bachem fixed merge-ort rejecting valueless rename settings like `merge.renames` by reading them with `repo_config_get_value()` instead of `repo_config_get_string()`.
- Elia Pinto posted a four-patch series using `size_t` when parsing and checking objects to avoid truncated sizes above 4GiB on platforms where `long` is 32 bits.
- Elia Pinto posted a companion series using `size_t` when receiving and verifying packs in unpack-objects, index-pack, and fsck pack verification.
- Phillip Wood noted that SIGBUS, SIGFPE, SIGILL and SIGSEGV are thread-directed and should not be blocked, and suggested a flag in the signal handler to guard against concurrent double-pops.
- Junio C Hamano decided to keep the test-number fix-up for the `^/` submodule URL syntax as a separate commit.
- Kristoffer Haugsbakk retracted his earlier claim about GenAI disclosure requirements in Documentation/SubmittingPatches.
- Harald Nordgren acknowledged a concern that CRLF line endings could make imap-send's header-end detection scan into the message body.
- Elia Pinto posted v3 of a series using `size_t` for object and buffer sizes in odb, diff, and convert.
- Elia Pinto posted v2 of a pack-objects patch using `off_t` for streamed large blob lengths, following Simon Richter's comment.
