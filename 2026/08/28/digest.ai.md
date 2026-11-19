# Git mailing list daily digest for 2026/08/28

## Long threads

### packfile recovery when a multi-pack-index names a removed pack

*Long thread (root 2026/08/18/22-34-04): 24 replies so far from 5 participants over 11 days; 1 message that day.*

This thread concerns the object database's packfile lookup path: when a geometric repack writes a new pack plus multi-pack-index and deletes the packs the new one subsumes, a concurrent reader can find an object reported as missing even though it is present, because the stale MIDX names a removed pack as the owner and the non-MIDX fallback skips all MIDX-covered packs. Elijah Newren's series fixes this by making `fill_midx_entry()` return a tri-state distinguishing "absent from the MIDX" from "present but owning pack unavailable", and by gating the recovery re-read on `OBJECT_INFO_SECOND_READ`. Today Elijah Newren asked for suggestions on alternate names for the tri-state `fill_midx_entry` function, proposing `fill_midx_entry_result` or `midx_fill_entry`; he also agreed that, with the QUICK-reader patch dropped, the code should directly `return MIDX_FILL_OWNER_UNAVAILABLE` rather than `goto owner_unavailable`, and accepted Jeff King's point that the comment rewrite was superfluous and should be kept to a minimum.

### commit: refuse to amend during conflict resolution

*Long thread (root 2026/08/26/05-21-21): 21 replies so far from 3 participants over 3 days; 8 messages that day.*

This series touches `builtin/commit.c` and `sequencer.c`, extending the long-standing refusal of `git commit --amend` and partial commits during conflict resolution -- previously only for merge and cherry-pick -- to `am`, `revert`, and rebase conflict stops, via a new `sequencer_ongoing_operation()` helper. Today Elijah Newren posted [PATCH v3 0/5], adding two preparatory patches: renaming `FROM_REBASE_PICK`/`is_from_rebase()` to reflect that they denote empty commits, and allowing a partial commit when a rebase pick becomes empty; the amend error for a dropped empty commit was tweaked to "The now-empty commit has been dropped -- cannot amend." Junio C Hamano reviewed patches 2/5, 3/5, and 5/5 approvingly ("OK. Looking good.", "Much less awkward than the previous round", "Looks quite thorough"), questioned whether `is_from_rebase_empty()` conveys the distinction between a commit that becomes empty and picking an empty commit, and Elijah Newren proposed the alternative names `FROM_REBASE_NOW_EMPTY` and `is_from_rebase_now_empty()`.

### checkout/switch/restore code refactoring

*Long thread (root 2026/08/27/18-29-00): 14 replies so far from 4 participants over 2 days; 8 messages that day.*

This is a refactoring of `builtin/checkout.c`, `checkout.c`, and `checkout.h`: the three front-ends `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` currently funnel into a monolithic `checkout_main()` choke point, and the series repartitions them to dispatch directly to `checkout_branch()` or `checkout_paths()`. Today Junio C Hamano posted [PATCH 0/8] "More sensible checkout/switch/restore code refactoring", an early rough draft he does not plan to merge even to `seen`, comprising explicit `cb_option` passing, branch-name validation in `checkout_branch()`, stage/merge validation in `checkout_paths()`, extracted helpers, restructured entrypoints removing `checkout_main()`, line wrapping, and moving `post_checkout_hook()` to `checkout.c`. He flagged an editor slip in patch 7/8 (`opts->new_new_branch_force` should be `opts->new_branch_force`) and noted that patch 8/8 is not a good libification illustration: `post_checkout_hook()` should instead take a `struct repository *repo` parameter and learn the hash algorithm from that repository instance.

## The day in brief

Junio C Hamano posted a rough-draft checkout/switch/restore refactoring in `builtin/checkout.c`, Elijah Newren sent v3 refusing `git commit --amend` and partial commits during am/revert/rebase conflict stops, and Yoichi NAKAYAMA's v4 worktree-repair fix for relative `.git` paths was slated for `next` after review as "looks very good to me".

## Notable threads

### worktree repair: detect relative path in .git file correctly

This fix in `worktree.c` addresses `git worktree repair` failing to convert relative cross-references to absolute paths when `worktree.useRelativePaths=false`, because `read_gitfile_gently()` always resolves to an absolute path. Today Yoichi NAKAYAMA posted v4, which introduces `read_gitfile_raw()` in `setup.c` to read the `.git` file contents without resolving them, refactors `read_gitfile_gently()` to call it, has `repair_gitfile()` use it with an explicit `is_git_directory()` check, and updates `t/t2406-worktree-repair.sh` to cover repair from both side and main worktrees. Junio C Hamano reviewed the iteration as "looks very good to me" and said he will mark it for `next` unless comments say otherwise in the next few days.

### format-patch: learn --[no-]range-diff-notes

Kristoffer Haugsbakk's series would have taught `git format-patch` options to tweak notes output in the range diff independently of patch notes, touching `builtin/log.c`, `revision.c`, `revision.h`, and `t/t3206-range-diff.sh`. Today Junio C Hamano questioned whether the design supports `--no-notes --range-diff-notes` (notes in the range-diff but not in patches), Kristoffer Haugsbakk confirmed it does not and proposed dropping the topic, and Junio C Hamano agreed to drop it.

### dir: fix negative pathspecs in 'git ls-files' and 'git add'

Diogo Castro's patch fixes `dir.c`'s pathspec matching, where the common prefix of positive pathspecs is stripped from all pathspecs including negative ones, so an exclude shorter than the common prefix degenerates to "" and matches everything (and can read out of bounds). Today he submitted the fix -- stripping the common prefix only from positive pathspecs in `do_match_pathspec()` -- with regression tests for both `git ls-files` and `git add`. Junio C Hamano questioned the approach, asking whether a command like `git ls-files -- a/b/c a/b/d !a/b/` should instead strip `a/b/` from all three pathspecs, and whether the common prefix should be computed across both positive and negative pathspecs.

### odb: write alternates at creation time

This series refactors how the object database handles alternates, moving from a `write_alternates()` callback used only at repository creation to writing alternates when the source creates the database, touching `odb/source*.c`, `odb/source.h`, `builtin/clone.c`, and `setup.c`. Today Toon Claes reviewed several patches, suggesting the `strvec.h` include be omitted in patch 6/8, asking why `fclose(in)` is not placed before the `out` label in patch 5/8, and asking why patch 8/8 does not use a lockfile. Junio C Hamano questioned whether the append-mode write to the alternates file in patch 6/8 should use the standard lockfile API to avoid a partially written file, or whether it does not matter since this happens only once at repository creation.

### http: add http.sslVerifyStatus to check stapled OCSP responses

Grayson Gordon's v6 patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests the OCSP stapled response, with per-URL config support and tests in the HTTP test suite. Today Grayson Gordon asked Junio C Hamano to clarify whether he prefers more thorough tests or succinct ones, and Junio C Hamano said succinct tests covering the essential bits would be great, asked which parts of the tests make Patrick uncertain, and noted the tests appear not to cover the case where the server does not explicitly say the certificate is still good.

## In brief

- Hardik Kumar posted an NFC patch fixing two typos: "fractionnal" to "fractional" in a comment in `versioncmp.c` and "similiarity" to "similarity" in the `test_description` of `t/t0022-crlf-rename.sh`.
