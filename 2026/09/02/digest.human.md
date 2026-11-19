# Git mailing list daily digest for 2026/09/02

## Long threads

### Convert build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread: 41 replies so far from 6 participants over 27 days; 3 messages that day.*

D. Ben Knoble's series converts the build-time `USE_NSEC` compile option into a runtime `core.useNanosec` setting, replacing `#ifdef` checks with runtime checks in `read-cache.c` and `statinfo.c` so distributed builds hit racy-git problems less often. Jeff King clarified that under `NO_NSEC` index nsec entries are faked as "0" via the `ST_CTIME_NSEC`/`ST_MTIME_NSEC` macros, making the setting a noop on such builds. Junio C Hamano asked whether a final reroll will use `ST_XTIME_NSEC(st)` to hide `NO_NSEC` differences; he remains worried that `repo_config_values()` can `BUG()` on a repository other than `the_repository` and suggested a safe default there, otherwise calling the series in great shape.

### Introduce the `receive-report` hook for `git-receive-pack`

*Long thread: 31 replies so far from 5 participants over 16 days; 3 messages that day.*

Karthik Nayak's series adds a `receive-report` hook that runs after the reference transaction completes but before the report reaches the client: it reads the pkt-line report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. Karthik agreed with Junio C Hamano that an `enum report_version` (v0/v2) fits the protocol-level difference better than a `bool detailed_report`, and defended the hook against the reference-transaction alternative, noting that no backend supports a post-commit rollback. Junio clarified his concern: when the hook reports failure, the pusher's remote-tracking branches reflect a failed push although the server already committed, so only a later fetch reconciles them.

### Refuse `git commit --amend` and partial commits during conflict resolution

*Long thread: 26 replies so far from 3 participants over 8 days; 5 messages that day.*

Elijah Newren's series makes `git commit --amend` and partial commits (`git commit <paths>`) die during conflict resolution in merge, cherry-pick, revert, am, or rebase, since recording a resolution captures the whole tree rather than rewriting HEAD. Phillip Wood reviewed v4: the `FROM_REBASE_NOW_EMPTY` rename and the reworded amend error for a dropped empty commit are clearer, and patch 4 looks good with `rebase_path_*()` accessors. He had mixed feelings about patch 2, which permits a partial commit when a rebase pick becomes empty, since `CHERRY_PICK_HEAD` and `.git/MERGE_MSG` are written at such stops and `git cherry-pick` still disallows them.

### Repartition checkout/switch/restore away from monolithic checkout_main()

*Long thread: 26 replies so far from 5 participants over 7 days; 3 messages that day.*

Junio C Hamano's eight-patch draft reworks cmd_checkout(), cmd_switch(), and cmd_restore() to dispatch directly to checkout_branch() or checkout_paths() instead of the monolithic checkout_main(). Karthik Nayak agreed that patch 6 makes patch 1's removal of cb_option coherent, and clarified that patch 3's changed guard is safe because --patch mode already rejects conflicting option combinations earlier in checkout_paths(). Junio confirmed those combinations were already rejected, answered "Perhaps." to an explanatory commit-message sentence, and said the series "merely scratches the surface" since a whole command like "git restore" may still be too coarse as reusable code.

## The day in brief

Three long-running series drew mostly approving review: the runtime `core.useNanosec` conversion, the `receive-report` hook, and the refusal of amend and partial commits during conflict resolution. Separately, a shallow-clone push optimization drew a reported multi-ref flaw, and a new patch keeps background `rerere gc` from killing a rebase.

## Notable threads

### Avoid sending the whole tree when pushing from a shallow clone

Elijah Newren's patch stops a shallow clone from pushing its entire toplevel tree by assuming the server has the shallow graft itself. Derrick Stolee reported that an AI-agent review found a multi-ref flaw: `append_reachable_shallow_grafts` treats shallow grafts as reachable without checking the remote's advertised refs, and he supplied a failing test. Newren countered that `push.negotiate=true` can also fail when a concurrent server repack defeats upload-pack's quick "have" check, and argued the multi-ref case is not corruption because receive-pack checks connectivity regardless of protocol.

### Keep a background `rerere gc` from killing a rebase

Thomas Bachem's patch makes `git rerere gc` warn and skip when `MERGE_RR.lock` is busy, since auto maintenance now runs it behind every commit and can collide with the sequencer's rerere call mid-rebase. Phillip Wood instead proposed disabling auto gc during rebase with `-c gc.auto=false -c maintenance.auto=false` and called the die in `do_pick_commit()` a bug. Bachem agreed to a two-patch v2: rerere returning an error on a busy lock while gc still warns and skips, plus the sequencer disabling auto gc.

### Fix out-of-bounds read when a negative pathspec is shorter than the prefix

Yannik Tausch's patch fixes an out-of-bounds read in `dir.c`: `match_pathspec_with_flags()` passed the common prefix derived only from positive pathspecs when matching negative ones, so a negative pathspec shorter than the prefix produced a negative matchlen. The fix passes 0 as the prefix for negative pathspecs and adds a regression test in `t/t6132-pathspec-exclude.sh`; the issue was disclosed to the Git security mailing list and confirmed not to require an embargo.

## In brief

- Junio C Hamano rejected Alessio Attilio's `hooks.allowNoVerify` proposal as a slippery slope; brian m. carlson agreed, noting hooks are trivially bypassable via `core.hooksPath=/dev/null`.
- Christian Couder posted a six-patch series adding an `early_scan_options()` sub-API to parse-options, fixing `--` detection in bisect and rev-parse and `--allow-unsafe-features` in fast-import; Junio welcomed it but prefers a future dry-run parser.
- Harald Nordgren reposted v4 of the `checkout -m` autostash refinement, where conflicts exit with status 1 and other failures 128; Junio called the second patch good.
- Jinbao Chen fixed a SIGSEGV in `git history split` when the parent tree object is missing, erroring with "unable to parse tree" instead of dereferencing NULL.
- Wolfgang Faust added a `--draft` option to `git imap-send` to set the IMAP `\Draft` flag; Junio affirmed the flag-first approach.
