# Git mailing list daily digest for 2026/08/27

## Long threads

### worktree add messages for ambiguous remote branch names

*Long thread: 32 replies so far from 6 participants over 20 days; 2 messages that day.*

Yoichi Nakayama posted v10 of his four-patch series improving the error and advice shown when `git worktree add` and `git checkout` cannot disambiguate a branch name matching multiple remote-tracking branches; the changes from v9 fix the function name in the first patch and change the type of `tracking_name_data.remote_names` in the second. Junio C Hamano reviewed v10, said it looks good to him, and stated he will mark it for `next` unless other comments arrive in a few days. Earlier, the series had grown a fourth commit making `git worktree add --guess-remote` treat multiple matches as an error instead of falling back to HEAD.

### packfile lookups racing a concurrent geometric repack

*Long thread: 23 replies so far from 5 participants over 10 days; 4 messages that day.*

Elijah Newren's series fixes objects being treated as missing when a geometric repack deletes a pack that a stale multi-pack-index still names as owner. Today Jeff King objected that `OBJECT_INFO_QUICK` is designed to accept false negatives, so `git mktree --batch` is wrong to use QUICK at all, and called the stale-packs mechanism a hack that triggers `SECOND_READ` for QUICK queries and defeats QUICK's purpose; he also flagged the `open_pack_index()` change as racy. Elijah Newren conceded the point, decided to drop patch 3/4's packfile.c changes entirely, and will instead send a small mktree change that stops passing `OBJECT_INFO_QUICK` so mktree recovers via the normal reprepare like other non-QUICK readers.

### commit refusing --amend and partial commits during conflict resolution

*Long thread: 14 replies so far from 3 participants over 2 days; 12 messages that day.*

Elijah Newren posted v2 of his three-patch series extending the refusal of `git commit --amend` and partial commits (`git commit <paths>`) beyond merge and cherry-pick to `am`, `revert`, and rebase conflict stops, rewording the empty-commit rebase errors and adding a `sequencer_ongoing_operation()` helper. Phillip Wood reviewed it, approving patch 3/3 and suggesting the sequencer's `rebase_path_stopped_sha()` helpers instead of hardcoded paths; Junio C Hamano questioned the reworded message and the `FROM_REBASE_PICK` naming. Newren said he will send a v3 incorporating the suggestions.

### builtin command `repo` parameter

*Long thread: 7 replies so far from 3 participants over 1 day; 9 messages that day.*

Hardik Kumar posted a patch replacing global `the_repository` uses with the previously-UNUSED `repo` parameter in `is_bare_repository()` call sites in `cmd_blame`, `cmd_gc`, `get_layout_bare`, `cmd_reset`, and `transport_push`, verified by the full test suite. Junio C Hamano countered that mixing `the_repository` and `repo` in one function is less consistent than the original and posted an illustrative patch replacing the `struct repository *` parameter of all `cmd_foo()` builtins with `bool has_repo`, asking for opinions; he also answered Ben Knoble that libification means factoring reusable logic out of `cmd_checkout()` rather than calling `main()` from elsewhere.

## The day in brief
Yoichi Nakayama's v10 worktree ambiguous-remote message series earned Junio C Hamano's approval and a stated intent to mark it for `next`. On the packfile race thread, Elijah Newren conceded Jeff King's objection to QUICK usage and will rework the mktree fix. Elijah Newren also posted v2 of the series refusing `git commit --amend` and partial commits during conflict resolution, drawing a favorable review from Phillip Wood.

## Notable threads

### sequencer honoring --empty when a fixup empties its target
Farid Zakaria posted v4 of the patch making `git rebase --autosquash --empty=drop` drop a commit that a `fixup!` or `squash!` empties, rebased onto master. V4 restricts dropping to commits that become empty, allows only the final fixup of a chain to drop the emptied commit, never drops when the chain is not preceded by a pick, and extends `commit_staged_changes()` to honor `--empty=drop` for a conflicted fixup resolved by undoing its target.

### branch -d protecting local upstreams
Elijah Newren reported a hole in Harald Nordgren's series protecting local branches from `git branch -d` when a surviving branch depends on them through an upstream chain: with branches A, B, C where B tracks A and C tracks B, `git branch -d A B` protected B but deleted A, leaving surviving B with a missing upstream, contradicting the patch's claim.

### die_for_incompatible_opts() for an unbounded number of options
Junio C Hamano posted v2 introducing `die_for_incompatible_opts()` to check an arbitrary number of mutually exclusive options and rewriting the existing opt2/opt3/opt4 variants on top of it. V2 adds the `va_end()` that Elijah Newren had flagged as missing and uses `va_arg(ap, int)` to distinguish bool values from the EOF sentinel.

## In brief
- Phillip Wood reviewed Harald Nordgren's `checkout -m` autostash refinement: he welcomed the conflict-result blank line before the branch-switch message but pushed back on gating the autostash retry on tracked local changes, which would require checking before every `unpack_trees()` call.
- Jeff King commented on Derrick Stolee's trace2 banned-die.h approach: banning `die()` catches only direct calls, and truly allocation-free trace2 would probably require a ground-up no-dependency rewrite.
- Yoichi Nakayama explained moving the `is_git_directory()` check out of `read_gitfile_raw()` into worktree.c for the worktree repair relative-path fix; Junio C Hamano confirmed the functional split is fine.
- Junio C Hamano posted v3 of the `you_still_use_that()` rewording, fixing "mailing list" and correcting two tests to look for "will be removed soon".
- Wesley Schwengle submitted v2 of the git-svn series silencing v1-layout migration noise on plain repositories and adding a `NO_GIT_SVN` Makefile knob to skip building and installing git-svn.
