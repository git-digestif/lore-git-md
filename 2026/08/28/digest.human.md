# Git mailing list daily digest for 2026/08/28

## Long threads

### packfile recovery when a multi-pack-index names a removed pack

*Long thread: 24 replies so far from 5 participants over 11 days; 1 message that day.*

This series addresses objects being treated as missing despite being present, due to a race with geometric repacking: a repack writes a new pack plus multi-pack-index and then deletes the subsumed packs, and a stale MIDX can name a removed pack as the owner of objects that survive elsewhere. Elijah Newren's v2 switched `fill_midx_entry()` to a tri-state and gated the recovery re-read on `SECOND_READ`. Today Elijah Newren asked for suggestions on alternate names for the tri-state `fill_midx_entry` function, proposing `fill_midx_entry_result` or `midx_fill_entry`; agreed that, with the QUICK-reader patch dropped, the code should directly `return MIDX_FILL_OWNER_UNAVAILABLE` rather than `goto owner_unavailable`; and accepted Jeff King's point that the comment rewrite was superfluous and should be kept to a minimum.

### commit: refuse to amend and partial commits during conflict resolution

*Long thread: 21 replies so far from 3 participants over 3 days; 8 messages that day.*

This series extends the long-standing refusal of `git commit --amend` during conflict resolution -- previously only for merge and cherry-pick -- to `am`, `revert`, and rebase conflict stops, and likewise refuses partial commits (`git commit <paths>`) during those operations, via a new `sequencer_ongoing_operation()` helper. Today Elijah Newren posted v3, adding two preparatory patches: renaming `FROM_REBASE_PICK`/`is_from_rebase()` to reflect that they denote empty commits, and allowing a partial commit when a rebase pick becomes empty, plus a tweaked amend error for a dropped empty commit. Junio C Hamano reviewed patches 2/5, 3/5, and 5/5 approvingly, questioned whether `is_from_rebase_empty()` conveys the distinction from picking an empty commit, and Elijah Newren proposed `FROM_REBASE_NOW_EMPTY`/`is_from_rebase_now_empty()` in response.

### checkout/switch/restore code refactoring

*Long thread: 14 replies so far from 4 participants over 2 days; 8 messages that day.*

This series repartitions the monolithic `checkout_main()` choke point in `builtin/checkout.c` so that `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` dispatch directly to `checkout_branch()` or `checkout_paths()`. Earlier discussion centered on whether builtins should keep a `struct repository *` parameter or take a `bool has_repo`, with Junio C Hamano arguing the change makes `cmd_foo()` implementations less error-prone and encourages factoring reusable helpers out of `builtin/`. Today Junio C Hamano posted the eight-patch series as an early rough draft he does not plan to merge even to `seen`, flagged an editor slip in patch 7/8 (`opts->new_new_branch_force` should be `opts->new_branch_force`), and noted that patch 8/8 is not a good libification illustration because `post_checkout_hook()` should instead take a `struct repository *` parameter.

## The day in brief

Elijah Newren posted v3 of the series refusing `git commit --amend` and partial commits during conflict resolution, incorporating renames and review feedback. Yoichi NAKAYAMA's v4 fix for `git worktree repair` detecting relative paths in the `.git` file was judged very good; Junio C Hamano said he will mark it for `next` unless further comments arrive. The proposed `format-patch --[no-]range-diff-notes` option was dropped after its interaction with `--notes` could not be defined simply.

## Notable threads

### format-patch: learn --[no-]range-diff-notes
Kristoffer Haugsbakk proposed teaching `format-patch` options to tweak notes output in the range diff independently of notes in the patches, motivated by wanting to turn off range-diff notes. Today Junio C Hamano questioned whether the design supports `--no-notes --range-diff-notes`, stated his preference against a new option whose interaction with `--notes` cannot be defined simply, Kristoffer Haugsbakk confirmed the use case is unsupported and proposed dropping the topic, and Junio C Hamano agreed to drop it.

### worktree repair: detect relative path in .git file correctly
Yoichi NAKAYAMA's patch fixes `git worktree repair` failing to convert relative cross-references to absolute paths when `worktree.useRelativePaths=false`, because `read_gitfile_gently()` always resolves to an absolute path; the fix introduces `read_gitfile_raw()` and has `repair_gitfile()` validate with `is_git_directory()`. Today Yoichi NAKAYAMA posted v4, refactoring `read_gitfile_gently()` to call the new function, clarifying the commit message, and updating `t/t2406-worktree-repair.sh` to cover repair from both side and main worktrees. Junio C Hamano reviewed it as "looks very good to me" and said he will mark it for `next` unless further comments arrive.

### dir: fix negative pathspecs in 'git ls-files' and 'git add'
Diogo Castro submitted a patch fixing negative pathspecs in `git ls-files` and `git add`: the common prefix of positive pathspecs is stripped from all pathspecs, so an exclude shorter than that prefix degenerates to "" and matches everything, and can read out of bounds. The fix strips the common prefix only from positive pathspecs in `do_match_pathspec()`, with regression tests. Today Junio C Hamano questioned the approach, asking whether a command like `git ls-files -- a/b/c a/b/d !a/b/` should instead strip `a/b/` from all three pathspecs, and whether the common prefix should be computed across both positive and negative pathspecs.

## In brief
- Toon Claes reviewed the `odb` alternates-at-creation-time series, suggesting placement and cleanup tweaks, and Junio C Hamano asked whether the append-mode write to the alternates file should use the lockfile API instead.
- Grayson Gordon asked Junio C Hamano to clarify a preference between Patrick's thorough `http.sslVerifyStatus` tests and succinct ones; Junio C Hamano asked which parts of the tests make Patrick uncertain.
- Hardik Kumar posted an NFC patch fixing typos in `versioncmp.c` ("fractionnal" to "fractional") and `t/t0022-crlf-rename.sh` ("similiarity" to "similarity").
