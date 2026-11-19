# Git mailing list daily digest for 2026/09/02

## Long threads

### Convert build-time USE_NSEC into runtime core.useNanosec

*Long thread (root 2026/08/07/11-56-23): 41 replies so far from 6 participants over 27 days; 3 messages that day.*

D. Ben Knoble's series converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration setting, replacing `#ifdef` checks with runtime checks in `read-cache.c` and `statinfo.c` and updating the racy-git documentation, so index-related commands are less likely to hit racy-git problems on supported filesystems. Jeff King clarified that under `NO_NSEC` the index stat nsec entries are faked as "0" via the `ST_CTIME_NSEC`/`ST_MTIME_NSEC` macros rather than `cfg->use_nanosec`, so on such builds `core.usenanosec` would be effectively a noop; he described a hypothetical rebuild sequence in which index values could appear up to one second older, but said his suggestion was only about simplifying the code and is not worth worrying about since `NO_NSEC` cannot be dropped entirely. Junio C Hamano asked whether a hopefully small and final reroll will follow that takes advantage of `ST_XTIME_NSEC(st)` hiding `NO_NSEC` build-time differences; he remains worried that code this deep in the callchain can `BUG()` on a repository that is not `the_repository` due to the use of `repo_config_values()`, and suggested adopting a safe default when `istate->repo` differs from `the_repository`, but otherwise considers the series in great shape.

### Introduce the receive-report hook for git-receive-pack

*Long thread (root 2026/08/18/07-55-55): 31 replies so far from 5 participants over 16 days; 3 messages that day.*

Karthik Nayak's series adds a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client, receiving the pkt-line encoded report on stdin with its stdout replacing the report sent to the user; a non-zero exit marks all references rejected. Karthik Nayak agreed with Junio C Hamano that an `enum report_version` (v0/v2) better captures that the `report()`/`report_v2()` difference is a protocol-level change than a `bool detailed_report` parameter, and said he would add it if he re-rolls the message-generation refactor. He also defended the hook against the objection that a reference-transaction hook would be an equally viable alternative: in GitLab's setup a successful `ref_transaction_commit()` is not the final step of a push, the transaction hook cannot express a ref push failure or send sideband messages, and rejecting at commit phase would require a post-commit rollback no backend supports; he will reroll with the rationale rewritten along these lines. Junio C Hamano clarified his remaining concern: when the hook reports failure, the pusher's local remote-tracking branches reflect a failed push even though the server already committed the ref transaction, so only the user's subsequent fetch can reconcile local state with the server.

### Refuse to amend or partially commit during conflict resolution

*Long thread (root 2026/08/26/05-21-21): 26 replies so far from 3 participants over 8 days; 5 messages that day.*

Elijah Newren's series makes both `git commit --amend` and partial commits (`git commit <paths>`) die during conflict-resolution stops of merge, cherry-pick, revert, am, and rebase, via a new `sequencer_ongoing_operation()` helper, while permitting a partial commit when a rebase pick becomes empty. Phillip Wood reviewed the v4 series: he said the renamed `FROM_REBASE_NOW_EMPTY`/`is_from_rebase_now_empty()` names are much clearer, agreed the reworded amend error for a now-empty rebase commit is clearer, and said the `--amend` refusal looks good now that it uses `rebase_path_*()` accessors rather than hard-coded paths. He expressed mixed feelings about allowing partial commits when a rebase pick becomes empty, noting that `CHERRY_PICK_HEAD` and `.git/MERGE_MSG` are written at such stops so the user can preserve the commit with `--allow-empty`, and that `git cherry-pick` still disallows partial commits for empty commits; he concluded he has no strong opinion either way but is happy with everything else.

### Repartition checkout/switch/restore code

*Long thread (root 2026/08/27/18-29-00): 26 replies so far from 5 participants over 7 days; 3 messages that day.*

Junio C Hamano's eight-patch series reworks `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` so they dispatch directly to `checkout_branch()` or `checkout_paths()` instead of funneling through the monolithic `checkout_main()`, moving option validation into the helpers and making `post_checkout_hook()` take a `struct repository *`. Karthik Nayak agreed that patch 6/8 (removing `checkout_main()`) makes sense together with patch 1/8, and clarified that in patch 3/8 the guard on the `--ours`/`--theirs`/`--force`/`--merge` incompatibility check changed because `checkout_paths()` is now also entered for `--patch` without a pathspec; he concluded this is safe because in patch mode each conflicting combination is already rejected earlier in `checkout_paths()`, and suggested a commit-message sentence explaining why. Junio C Hamano confirmed the patch-mode combinations were already rejected before the moved check, responded "Perhaps." to the commit-message suggestion, and stated that as a unit of reusable code an entire command like `git restore` is still too coarse for callers wanting finer-grained control from C, so the patches are only a starting point that scratches the surface.

## The day in brief
Christian Couder's early-scan parse-options series arrived with fixes for `git bisect`, `git rev-parse`, and `git fast-import` argument parsing, drawing a welcome but also a call for a properly refactored dry-run parser. Derrick Stolee's AI-agent review found a multi-ref push flaw in Elijah Newren's shallow-clone push optimization, sparking debate over `push.negotiate` as a backstop. Thomas Bachem's rerere-gc-versus-rebase fix evolved toward disabling auto gc during rebase, and Junio C Hamano signaled the `core.useNanosec` runtime conversion is in great shape pending a possible final reroll.

## Notable threads

### Avoid sending the whole tree when pushing from a shallow clone
Elijah Newren's patch avoids sending the entire toplevel tree when pushing from a shallow clone by assuming the server has the shallow graft itself rather than only its parents. Derrick Stolee reported that an AI-agent review of the patch found a flaw: in a multi-ref push, a shallow update attempted alongside another non-shallow branch can cause novel failures, because `append_reachable_shallow_grafts` treats shallow grafts as reachable from the refs being pushed rather than verifying they are reachable from the remote's advertised refs; he supplied a new t5538 test that passes before the patch but fails after it. Stolee argued the patch's assumption that non-advertised objects may exist on the server is not a reasonable default. Newren countered that `push.negotiate=true` can fail when a concurrent server repack makes upload-pack's QUICK "have" check lose the race, so a backstop is still needed, and stated the multi-ref case is not a corruption risk because receive-pack checks connectivity regardless of protocol, merely failing more refs than necessary.

### Standardize early option scanning in parse-options
Christian Couder posted a six-patch series introducing an "early-scan" sub-API (`early_scan_options()`) into parse-options to fix argument-parsing bugs where hand-rolled early scans mistake an option's value for the `--` separator, converting `git bisect`, `git rev-parse`, and `git fast-import` -- including fixing `git fast-import --depth 5 --allow-unsafe-features` silently ignoring the option. Junio C Hamano welcomed the series but said he fails to see much practical value in the bisect and rev-parse fixes while the fast-import fix "may be a very good thing", and stated a long-term preference for a properly refactored parse-options prescan such as a "dry-run" mode, asking that the interim solution at least take the same options array.

### Keep a background rerere gc from killing a rebase
Thomas Bachem posted a patch so that `git rerere gc` warns and skips when `MERGE_RR.lock` is busy instead of dying, which since 2.54's geometric maintenance strategy can kill an in-progress rebase when the background `git maintenance run --auto` collides with the sequencer's `repo_rerere()`. Phillip Wood proposed instead disabling gc during rebase by passing `-c gc.auto=false -c maintenance.auto=false` in the sequencer's commit, merge, and exec paths, and making `setup_rerere()` return an error on a busy lock. Bachem agreed to a v2 of two patches -- rerere returning an error on a busy lock with gc still warning and skipping, plus the sequencer disabling auto gc -- which he would hold until Junio C Hamano and Patrick Steinhardt weigh in.

### Reserve exit status 1 for conflicts in stash and checkout -m
Harald Nordgren reposted v4 of the series refining the `checkout -m` autostash fallback: conflicts now exit with status 1 like merge-tree while other failures exit 128, and the autostash apply helpers use the `enum stash_apply_result` return value, claiming conflicts only when `git stash apply` actually reported them; a second commit separates the autostash conflict advice from the branch-switch message. Junio C Hamano suggested a small `do_stash_apply()` helper for v4 1/2 but otherwise marked it "Good", and said v4 2/2 "looks good to me".

### Fix out-of-bounds read with negative pathspecs in dir.c
Yannik Tausch submitted a patch fixing an out-of-bounds read in `dir.c`: `match_pathspec_with_flags()` passed the common prefix derived solely from positive pathspecs when matching negative pathspecs, so a negative pathspec shorter than the prefix produced a negative matchlen and a heap-buffer-overflow; the fix passes 0 as the prefix when matching negative pathspecs and adds a regression test in `t/t6132-pathspec-exclude.sh`.

## In brief
- Alessio Attilio posted v4 of a `hooks.allowNoVerify` configuration to control `--no-verify`; Junio C Hamano rejected the proposal as a slippery slope, and brian m. carlson agreed, noting hooks can be bypassed via `core.hooksPath=/dev/null`.
- **dir: common prefix among positive pathspecs**: Yannik Tausch proposed fixing `common_prefix_len()` returning zero when an exclude pathspec comes first; Junio C Hamano objected to the struct-by-value return and proposed an output parameter instead.

- Jinbao Chen fixed a SIGSEGV in `git history split` when the parent commit's tree object is missing, erroring with "unable to parse tree <oid>".
- Jeff King explained that porcelain commands allowing a branch named `@` is not a bug -- bare `@` is legal in a ref component for backwards compatibility -- and suggested treating it like `HEAD` in porcelain commands.
- Jeff King posted patches to use the system asciidoctor package in CI instead of the pinned 1.5.8 gem; Patrick Steinhardt agreed old-version checking is unnecessary.
- Patrick Steinhardt posted v2 of the series stopping in-memory ODB source registration, adapting `cache_tree_fully_valid()` to take a `struct index_state`.
- Junio C Hamano flagged a target-name inconsistency between Johannes Schindelin's two patches for using Rust in Windows CI, where UCRT64 maps to different Cargo targets in the Makefile and the shell script.
- Wolfgang Faust added `--draft` to `git imap-send` to set the IMAP `\Draft` flag; Junio C Hamano affirmed starting with a flag before adding configuration.
- **Submodule leak fix**: Jeff King posted patches making repo_clear() idempotent and freeing URL when submodule setup fails, fixing a Coverity-noticed leak. Patrick Steinhardt endorsed a self-contained parsed object pool.

- Patrick Steinhardt changed `cache_tree_fully_valid()` to accept a `struct index_state`, dropping its dependency on `the_repository`.
- Yuvraj Singh Chauhan moved `grafts_keep_true_parents` into `repo_config_values` with a getter, and added `t/t5300-pack-object.sh` tests for `--keep-true-parents`.
