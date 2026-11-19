# Git mailing list daily digest for 2026/09/14

## Long threads

### receive-report hook in git-receive-pack

*Long thread (root 2026/08/18/07-55-55): 57 replies so far from 7 participants over 28 days; 1 message that day.*

Karthik Nayak's series adds a `receive-report` hook to `builtin/receive-pack.c` that runs after the reference transaction completes but before the report reaches the client: it receives the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. Earlier, v10 fixed a leak when reassigning command strings in `override_cmds_error()`. Today, Junio C Hamano queued a "receive-pack: coccinelle fix" on top of the v10 series before it goes to `next`, dropping the nullness check of `cmd->error_string_owned` before `FREE_AND_NULL`, which is safe to call on NULL.

### auto maintenance once at the end of a sequence

*Long thread (root 2026/09/04/07-53-44): 30 replies so far from 5 participants over 11 days; 3 messages that day.*

Thomas Bachem's three-patch series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, calling it from `builtin/rebase.c` and `builtin/revert.c`, and passes `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS` so background maintenance cannot race the sequence for `MERGE_RR.lock` or delete open packs. Today, Bachem accepted Phillip Wood's rewording of the 3/3 commit message and explained the new test scenario, accepted Patrick Steinhardt's doc wording for `git_config_append_parameter()` while keeping a sentence on NULL values, and took Patrick's 1/3 text with corrections -- the sequencer spawns `git commit` on `--continue`, and the apply backend is `git am`. He asked Patrick to choose between the sequencer and the two builtins for the maintenance call and will reroll accordingly.

### consistent ref storage format terminology

*Long thread (root 2026/09/04/10-36-01): 27 replies so far from 4 participants over 11 days; 4 messages that day.*

Patrick Steinhardt's 13-patch series renames the inconsistent options, environment variables and config keys for the reference storage backend -- `--ref-format=`, `GIT_REFERENCE_BACKEND`, `init.defaultRefFormat` -- to "ref storage format" names like `--ref-storage-format=` and `GIT_REF_STORAGE_FORMAT`, keeping old names as hidden aliases and accepting `format://payload` URIs. Today, Karthik Nayak questioned whether hidden `OPT_ALIAS()` aliases make sense but called v3 good; Kaartic Sivaraam also approved v3 while asking for tests of the deprecated variants; Junio C Hamano wants more thought on `OPT_ALIAS()` but otherwise finds the series in very good shape.

### cut-and-pasteable advice squelch hints

*Long thread (root 2027/08/29/00-49-58): 23 replies so far from 4 participants over 6 days; 3 messages that day.*

Vsevolod Myalitsin's series adds a configuration scope hint to advice settings so the `defaultBranchName` advice shown at `git init` points at a scope affecting future repositories. Today, Jeff King suggested recommending `--global` for all advice messages, with per-repo hints as a later exception, and Junio C Hamano agreed; Junio then posted `[PATCH v5 1/1]` adding `--global` to the `git config set` command in the advice squelch message in `advice.c`, updating 11 test files and crediting Vsevolod Myalitsin and Jeff King.

## The day in brief
Junio C Hamano posted a v5 making every advice squelch hint cut-and-pasteable with `--global`, and queued a coccinelle fix on top of the v10 receive-report hook series. New series proposed an incremental connectivity check with large reported speedups and a fix for a `git repack -d` race that can delete a pack whose objects were never copied.

## Notable threads

### repack race with a mid-run ".keep" file
Qin ShiCheng posted a six-patch series fixing a race where `git repack -d` deletes a pack whose objects were never copied because a `.keep` appeared between repack's scan and pack-objects' rescan -- a production incident that left a ref pointing at a missing commit. receive-pack now deletes only `.keep` files carrying its own message, and repack passes its startup snapshot of kept packs to every spawned pack-objects via the new `--keep-pack-from-file=<file>` instead of `--honor-pack-keep`. Supporting changes mark `--keep-pack` entries kept-open under `--stdin-packs=follow`, invalidate the kept-pack cache in the cruft walk, sort the kept-pack list for binary lookup, and refuse pack names with newlines.

### incremental connectivity check in rev-list
Kristofer Karlsson posted a two-patch series adding an opt-in incremental connectivity check behind `transfer.connectivityCheck=incremental`, run inside the rev-list subprocess via a new `--verify-trees-incremental` flag, with benchmarks reporting up to ~190x speedup and much lower RSS for small pushes against a 3.4 GB boundary closure. Junio C Hamano called the benchmarks "exciting" but asked whether `is_promisor_object()` in `verify_blob()` comes too late and whether `get_commit_tree_oid()` can return NULL in `verify_commit_tree()`. Karlsson answered that `--exclude-promisor-objects` sets `repo->fetch_if_missing = 0`, confirmed the NULL-tree oversight, and will add a `die()` NULL check at both call sites plus a blob test.

### --force-if-includes rejects non-branch pushes
Tyler Cipriani posted v4 making `check_if_includes_upstream` resolve the peer ref with `RESOLVE_REF_READING` so all non-branch ref pushes are rejected under `--force-if-includes`, adding a tag-push test. D. Ben Knoble flagged that this breaks pushing `<TAG>:hotfix` with `push.useForceIfIncludes=true`; Cipriani confirmed the breakage, noting the check is already a false pass today because it consults the same-named local branch's reflog, not the pushed tag. He also reported that a fast-forward push fails under `--force-with-lease --force-if-includes` when the reflog is expired, and said v5 will touch other functions in `remote.c`.

### git var idents, signing key, multiple args, -z
Andrew Pleeter posted v8 teaching `git var` to expose broken-down idents (`GIT_AUTHOR_NAME` and friends), `GIT_SIGNING_KEY`, multiple variable arguments and `-z`. After Junio C Hamano noted v7's docs claimed exit code 1 for any missing value while the code only did so for single-variable queries, v8 prints `VARIABLE=value` pairs for multi-variable output (or `VARIABLE\nvalue\0` with `-z`), repeats multi-valued variables, omits unset ones while still exiting 1, and updates `Documentation/git-var.adoc` and `t/t0007-git-var.sh`.

### rerere waits for MERGE_RR.lock, skips at conflicts
Thomas Bachem posted v4 of a two-patch series: `setup_rerere()` waits up to `rerere.lockTimeout` (default 1000 ms) for `MERGE_RR.lock` before failing, and `git rerere gc` skips its run when the lock is held; the six callers that stop at a conflict warn and go on without rerere via `RERERE_SKIP_LOCKED`, while `git commit`, `git am --continue` and the `rerere clear` from am and rebase still fail after the wait.

## In brief
- Yannik Tausch posted v4 fixing pathspec prefix handling in `dir.c` when an exclude item leads: excludes match full pathnames and the prefix comes from the first non-exclude item, preserving the directory-walk optimization.
- Patrick Steinhardt approved Ariel Keselman's v3 skipping the packed-refs lock for root ref deletion in the files backend; Junio C Hamano will queue it for `next`.
- Johannes Schindelin announced Git for Windows 2.56.0-rc0, dropping Windows 8.1 (`git.exe` moves from `/mingw64/bin` to `/ucrt64/bin`) and fixing the v2.55.0(5) installer issue that overwrote external OpenSSH choices.
- Sean Whitton reported that `git status --ignored` with a pathspec matches partial names, e.g. `ba` matching ignored directory `bar`.
- **merge-ll re-roll plans**: Jeff King will add a fail-fast size check for external merge results, likely as a separate patch, re-roll patch 2 with distinct write/close die() calls, and credit Michal Koutný as Reported-by.

- Siddharth Asthana and Pablo Sabater applied as co-mentors for the parse-options and libification Outreachy projects; Kaartic Sivaraam approved both, giving each project two mentors.
- Todd Zullinger posted v2 converting `git-pack-refs` and `git-refs` docs to backtick quoting, and agreed to fix a commit message that mislabeled options as "configuration key terms".
- Harald Nordgren posted v2 dropping the no-op `brew link --force gettext` from `ci/install-dependencies.sh` since Homebrew's gettext stopped being keg-only in 2020; Patrick Steinhardt noted the cited pipeline did not test macOS, and Nordgren pointed to an earlier run that did.
- Yoichi Nakayama posted v2 adding `repair` to `git worktree` completion in the bash, zsh and tcsh scripts, changing only the commit message.
- Junio C Hamano retracted his suggestion to use `die_for_incompatible_opt3()` in `range-diff.c`, saying `die()` there would be a layering violation and the `error()`-based `--matched-only` conflict check is correct.
- Beat Bolli defended the imap-send OpenSSL 4.1 patch: `ASN1_STRING_length()` is deprecated since 4.1 and the build fails under `DEVELOPER=1`.
- Tanishq Singh replaced `test -f`/`-d`/`! -e` with `test_path_is_*` helpers in `t/t7610-mergetool.sh`; Junio C Hamano objected that `! test_path_is_file` is wrong for asserting absence and `test_path_is_missing()` should be used.
