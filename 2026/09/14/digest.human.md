# Git mailing list daily digest for 2026/09/14

## Long threads

### receive-report hook in `builtin/receive-pack.c`

*Long thread: 57 replies so far from 7 participants over 28 days; 1 message that day.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client: the hook receives the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. Earlier revisions fixed a BUG() when no report was requested and a leak when reassigning command strings. Today Junio C Hamano queued a "receive-pack: coccinelle fix" patch on top of the v10 series, removing the nullness check of `cmd->error_string_owned` before `FREE_AND_NULL` in `override_cmds_error()` in `builtin/receive-pack.c`, since `FREE_AND_NULL` is safe to call on NULL.

### sequencer auto maintenance timing

*Long thread: 30 replies so far from 5 participants over 11 days; 3 messages that day.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, and passes `maintenance.auto=false` to sequencer-spawned commit, merge and exec commands via `GIT_CONFIG_PARAMETERS` so background maintenance cannot race the sequence for `MERGE_RR.lock` or delete open packs. Today Bachem accepted Phillip Wood's rewording of the 3/3 commit message and explained the new test scenario in `t/t3418-rebase-continue.sh`, accepted Patrick Steinhardt's doc wording for `git_config_append_parameter()` while keeping a sentence on NULL values, and asked Steinhardt to choose between calling auto maintenance at the end of `pick_commits()` plus the single-pick return paths or in the two builtins, saying he will reroll accordingly.

### unifying ref storage format terminology

*Long thread: 27 replies so far from 4 participants over 11 days; 4 messages that day.*

Patrick Steinhardt's 13-patch v3 series renames the inconsistent ref storage format interfaces -- `--ref-format=`, `GIT_REFERENCE_BACKEND`, `init.defaultRefFormat` and friends -- to consistent `ref-storage-format` equivalents across init, clone, refs, submodule, rev-parse and help, keeping old names as hidden aliases and letting `--ref-storage-format=` accept `format://payload` URIs. Today Karthik Nayak questioned whether hidden `OPT_ALIAS()` entries make sense but said the v3 changes look good, Kaartic Sivaraam wondered whether tests should also cover the deprecated variants, and Junio C Hamano said he would like more thought on the `OPT_ALIAS()` question but otherwise found the series in very good shape.

### advice squelch hints with `--global`

*Long thread: 23 replies so far from 4 participants over 6 days; 3 messages that day.*

Vsevolod Myalitsin's series addresses the `defaultBranchName` advice, whose suggested `git config set advice.defaultBranchName false` writes to the local scope and so only affects the repository where the advice was already shown. Today Jeff King suggested simply recommending `--global` in all advice squelch messages, with per-repo exceptions getting a hint flag later, Junio C Hamano agreed with the simplicity, and Hamano posted [PATCH v5 1/1] "advice: give cut-and-pasteable advice to squelch", adding `--global` to the `git config set` command in the advice message across `advice.c` and 11 test files.

## The day in brief
Junio C Hamano queued a coccinelle fix on top of the v10 receive-report hook series and posted a v5 adding `--global` to advice squelch hints. New series arrived for an incremental connectivity check with reported ~190x speedups and for fixing a `git repack -d` race that can delete packs whose objects were never copied.

## Notable threads

### repack race with mid-run `.keep` files
Qin ShiCheng posted a 6-patch series fixing a production-hit race where `git repack -d` deletes a pack whose objects were never copied: a concurrent push can link a `.keep` onto an existing pack between repack's initial scan and pack-objects' rescan, so pack-objects leaves the objects out and repack deletes the pack anyway. The series makes receive-pack remove only `.keep` files it installed and has repack hand pack-objects its startup snapshot of kept packs via a new `--keep-pack-from-file=<file>` option instead of `--honor-pack-keep`, plus fixes for the cruft walk's kept-pack cache and binary lookup of kept packs.

### incremental connectivity check via rev-list
Kristofer Karlsson posted a two-patch series adding an opt-in incremental connectivity check gated behind `transfer.connectivityCheck=incremental`, running inside the rev-list subprocess via a new internal `--verify-trees-incremental` flag, with benchmarks reporting up to ~190x speedup and much lower RSS for small pushes against a 3.4 GB boundary closure. Junio C Hamano called the benchmarks "exciting" but raised two concerns on patch 2/2 -- a possibly too-late `is_promisor_object()` call in `verify_blob()` and a possible NULL from `get_commit_tree_oid()` passed to `tree_map_add()` -- and Karlsson confirmed both, explaining the promisor path is safe because `--exclude-promisor-objects` sets `fetch_if_missing = 0`, and said he will add a blob test and a NULL check with `die()`.

### `--force-if-includes` rejects non-branch sources
Tyler Cipriani posted v4 of a two-patch series making `--force-if-includes` reject pushes whose source ref is not a branch: `check_if_includes_upstream` now unconditionally resolves the peer ref with `RESOLVE_REF_READING`, so tag pushes are rejected too. D. Ben Knoble flagged that this breaks his workflow of pushing `<TAG>:hotfix` with `push.useForceIfIncludes=true`; Cipriani confirmed the break, noting the check today consults the same-named local branch's reflog rather than the pushed tag, and reported a related bug where a fast-forward push fails under `--force-with-lease --force-if-includes` when the reflog is expired, to be addressed in v5.

## In brief
- Andrew Pleeter posted v8 of `git var`, which now prints `VARIABLE=value` pairs for multi-variable queries (repeated entries for multi-valued variables, `VARIABLE\nvalue\0` with `-z`), omits unset variables while still exiting 1, fixing the ambiguity Junio C Hamano flagged in v7.
- Yannik Tausch posted v4 of a two-patch `dir` series fixing pathspec prefix handling with leading exclude items: exclude items now match against full paths (fixing an incorrect match and an out-of-bounds read), and the common prefix is derived from the first non-exclude item to preserve the directory-walk optimization.
- Thomas Bachem posted v4 of a two-patch rerere series: `setup_rerere()` waits up to `rerere.lockTimeout` for `MERGE_RR.lock` (with `git rerere gc` skipping instead), and the six callers that stop at a conflict warn and go on when the lock stays busy.
- Junio C Hamano said he will queue Ariel Keselman's v3 patch skipping the packed-refs transaction for root-ref deletions in the files backend and mark it for `next`.
- Johannes Schindelin announced Git for Windows 2.56.0-rc0, which drops Windows 8.1 support (moving `git.exe` from `/mingw64/bin` to `/ucrt64/bin`) and bundles Git v2.56.0-rc0, Git LFS v3.8.0, cURL v8.22.0 and OpenSSL v3.5.8.
