# Git mailing list daily digest for 2026/08/27

## Long threads

### worktree add messages for ambiguous remote branch names

*Long thread (root 2026/08/08/08-21-41): 32 replies so far from 6 participants over 20 days; 2 messages that day.*

Yoichi NAKAYAMA posted v10 of the four-patch series improving the error and advice shown when `git worktree add` (and `git checkout`) hits a branch name matching multiple remote-tracking branches; the changes from v9 fix the function name in 1/4 and the type of `tracking_name_data.remote_names` in 2/4. The series, which also makes `git worktree add --guess-remote` treat multiple matches as an error, previously drew Junio C Hamano's "Will queue" on v7 and his reviews of v8 and v9; today Junio C Hamano said v10 looks good to him and that he will mark it for `next` unless other comments are received in a few days.

### packfile lookups racing a concurrent geometric repack

*Long thread (root 2026/08/18/22-34-04): 23 replies so far from 5 participants over 10 days; 4 messages that day.*

Elijah Newren's v2 series addresses objects treated as missing despite being present when a geometric repack deletes packs that a stale multi-pack-index still names as their owner. Today Jeff King argued that `OBJECT_INFO_QUICK` is designed to accept false negatives, that `git mktree --batch` is wrong to use QUICK at all, and that the stale-packs mechanism triggering `SECOND_READ` for QUICK queries defeats QUICK's purpose; Elijah Newren conceded, decided to drop patch 3/4's `packfile.c` changes entirely, and will instead send a small mktree change that stops passing `OBJECT_INFO_QUICK` so mktree recovers via the normal reprepare like other non-QUICK readers. Jeff King also noted the mktree per-tree leak went uncaught because no test exercises `mktree --batch` with multiple inputs, and suggested adding basic directed tests to t1010.

### refusing commit --amend and partial commits during conflict resolution

*Long thread (root 2026/08/26/05-21-21): 14 replies so far from 3 participants over 2 days; 12 messages that day.*

Elijah Newren posted v2 of the three-patch series extending the long-standing refusal of `git commit --amend` during conflict resolution (previously only merge and cherry-pick) to `am`, `revert`, and `rebase` conflict stops, and adding refusal of partial commits (`git commit <paths>`) during the same operations; patch 1 rewords the empty-commit rebase errors. Phillip Wood reviewed, approving 3/3 and calling the cover letter a sound UI improvement, while Junio C Hamano questioned the 1/3 wording and suggested renaming `FROM_REBASE_PICK` to reflect that it denotes an empty-pick situation; Elijah Newren said he will send a v3 incorporating the suggestions.

### dropping the repository parameter from builtin commands

*Long thread (root 2026/08/27/18-29-00): 7 replies so far from 3 participants over 1 day; 9 messages that day.*

Hardik Kumar posted a patch replacing the global `the_repository` argument to `is_bare_repository()` with the previously-UNUSED `repo` parameter in `cmd_blame`, `cmd_gc`, `get_layout_bare`, `cmd_reset`, and two sites in `transport_push`, verified by running the entire test suite. Junio C Hamano countered that using `the_repository` and `repo` interchangeably in one function is less consistent than the original, and posted an illustrative patch replacing the `struct repository *` parameter of all `cmd_foo()` builtins with `bool has_repo`; he also answered Ben Knoble that libification means factoring the reusable part of `cmd_checkout()` into routines shared with `cmd_switch()`, not calling `main()` from elsewhere.

## The day in brief
Junio C Hamano said the v10 worktree-add ambiguity series looks good and will be marked for `next` absent further comments, Elijah Newren conceded Jeff King's point that `mktree --batch` should not use `OBJECT_INFO_QUICK` and will rework that patch, and he posted v2 extending `git commit` refusals during conflict resolution to `am`, `revert`, and `rebase`.

## Notable threads

### sequencer honoring --empty when a fixup empties its target
Farid Zakaria posted v4 of the patch making `git rebase --autosquash --empty=drop` drop a commit that a `fixup!`/`squash!` empties instead of halting with an amend-refusal message; v4 restricts dropping to commits that become empty, allows only the final fixup of a chain to drop the emptied commit, and reports the drop via a new `PICK_RESULT_DROPPED_HEAD` enum member.

### branch -d protecting local upstreams
Elijah Newren reported that Harald Nordgren's series protecting local branches from `git branch -d` when a surviving branch depends on them through a local upstream chain fails for indirect upstreams when multiple branches are deleted together: with B tracking A and C tracking B, `git branch -d A B` protected B but deleted A, leaving the surviving B with a missing upstream.

### trace2 stop allowing die()

Derrick Stolee's v2 series bans `die()` and allocation helpers from the trace2 API via a new `banned-die.h` header. Jeff King called the approach probably the best practical one for `die()` specifically, but warned it is whack-a-mole since banning direct calls misses transitive ones, and that all of strbuf would also need to be off-limits. He suggested avoiding allocations in trace2 would probably require a ground-up no-dependency rewrite.

### checkout -m autostash fallback refinement
Harald Nordgren's two-patch series refines `checkout -m` autostash retries and visually separates autostash conflict advice from the branch-switch message. Phillip Wood welcomed the message separation but pushed back on only autostashing when tracked local changes exist, arguing it would require checking for local changes before the first `unpack_trees()` call on every `git checkout -m`, trading an occasional unnecessary stash for that cost.

### die_for_incompatible_opts() for an unbounded number of options
Junio C Hamano posted v2 introducing `die_for_incompatible_opts()` to check an arbitrary number of mutually exclusive options, rewriting the existing opt2/opt3/opt4 variants on top of it; Elijah Newren had caught a missing `va_end()` in the earlier version, which v2 adds, and Jeff King suggested `LAST_ARG_MUST_BE_NULL` in the header to guard against a forgotten terminator.

## In brief
- Wesley Schwengle submitted v2 of patches stopping git-svn's v1-layout migration noise on plain repositories and adding a `NO_GIT_SVN` Makefile knob.
- Yoichi Nakayama and Junio C Hamano discussed the commit message for `read_gitfile_raw()` in worktree repair; Junio confirmed the functional split between the two functions is fine.
- Karthik Nayak reviewed Patrick Steinhardt's series making object database consistency checks pluggable, finding only small nits and questions.
- Junio C Hamano posted v3 of the `you_still_use_that()` rewording, fixing "mailing list" and correcting two tests to look for "will be removed soon".
- Ben Knoble confirmed the French git-checkout translation's "n'" in "s'il n'y a" is expletive rather than negation, and CC'd the translators to get it corrected.
- Kristoffer Haugsbakk acknowledged a whatchanged user's alias workaround and reminded them to Reply All so the response reaches the mailing list.
- Wesley Schwengle posted the `NO_GIT_SVN` Makefile patch that skips building and installing git-svn and implies `NO_SVN_TESTS`.
- Farid Zakaria noted that v4 of the sequencer --empty patch appears attached to another thread on lore.kernel.org.
- Kristoffer Haugsbakk posted a response regarding the kh/format-patch-range-diff-notes topic.
