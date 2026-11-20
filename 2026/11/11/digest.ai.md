# Git mailing list daily digest for 2026/11/11

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 72 replies so far from 5 participants over 54 days; 6 messages that day.*

Harald Nordgren's v13 series separates which refs to fetch from where they are stored via `remote.<name>.fetchRefmap` and `--limited-fetch`, and adds a sixth commit resolving `@{push}` through the refmap. Today he asked for hopefully final reviews. Phillip Wood criticized the cover letter: across 12 files changed it only restates the subject without explaining the problem, design, trade-offs, or discarded alternatives. Junio C Hamano agreed the criticism was fair, said he understood the first three patches, called patches 4 and 5 ergonomic icing on the cake, said he was not interested in patch 6, and noted the topic is holding up another author's topic. D. Ben Knoble asked whether patch 6's `@{push}` failure could be claimed as a bugfix belonging with the first three patches; Junio agreed that making `@{push}` take `remote.<name>.fetchRefmap` into account is a logical part of that work.

### repo: add path keys to `git repo info`

*Long thread (root 2026/07/16/01-21-31): 60 replies so far from 6 participants over 119 days; 3 messages that day.*

K Jayatheerth's GSoC series adds `path.*` keys to `git repo info` for toplevel, superproject, hooks, index, prefix, and cdup. Today he agreed to drop the `path.hooks` patch after Phillip Wood argued it would encourage an outdated interface that ignores config-based hooks, and to drop `path.cdup` as adding nothing over `path.working-tree.relative`; he also agreed to avoid hyphens in the superproject key name after Junio C Hamano noted Git generally avoids them. Phillip Wood acknowledged the plan and said he looked forward to v9.

### branch: let --delete-integrated find squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 55 replies so far from 6 participants over 44 days; 2 messages that day.*

Harald Nordgren's series adds `--delete-integrated` to `git branch`, deleting branches whose changes landed via squash or rebase merges and naming the landing commit. Jasper Mayone reviewed v6 patch 4/4: he flagged that `repo_read_index()` in `merge_keeps_upstream()` runs for each changed file at each candidate commit and still calls `clear_skip_worktree_from_present_files()`, which re-walks the index in a sparse checkout; noted the patch answers only half of Junio's earlier question because attributes now come from the current worktree's index rather than the upstream commit; and proposed a test where the merge does not leave upstream unchanged. He concluded that with those points the patch looks good to him.

### worktree: treat a branch being rebased or bisected as checked out

*Long thread (root 2026/10/21/17-33-36): 33 replies so far from 8 participants over 22 days; 2 messages that day.*

Jasper Mayone's series makes `git worktree remove` refuse worktrees mid-operation, documents reading `%(worktreepath)` safely, names the worktree rebasing or bisecting a branch, and stops `git branch` from marking that branch with "+". Today he posted v5: patch 1 rewords the `--force` cases as complete sentences, patch 3 uses a map in `branch.c` so `branch_checked_out_worktree()` is one lookup, new patch 4 adds `%(currentworktree)`, and patch 5 uses it in the default `git branch` format. Ben Knoble replied that the new wording in patch 1 is also OK by him.

### t: add "test-tool genbytes" to avoid slow tr(1) on macOS

*Long thread (root 2026/11/07/14-59-22): 30 replies so far from 5 participants over 5 days; 5 messages that day.*

Kristofer Karlsson's series adds `test-tool genbytes <char> [<count>]` so tests stop piping `genzeros` through macOS's slow `tr(1)`. Patrick Steinhardt reviewed v3's single-character argument and concluded "this looks great". Junio C Hamano pointed out that merging down would reveal a new `genzeros | tr` use in `t/t1600-index.sh` and suggested rebasing to catch it. Karlsson posted v4 rebased on latest master, converting that new large callsite alongside the existing ones in t0613, t4205, t5300, t7006, and t9850.

## The day in brief
The `log --follow` rename-following performance regression drew benchmarks showing v5 still far slower than the pre-regression behavior, with Patrick Steinhardt proposing a quickly landable disable knob and Junio C Hamano endorsing an opt-in fresh start. Harald Nordgren's fetchRefmap v13 series received review, including cover-letter criticism and agreement that the `@{push}` fix belongs with the core work. Several documentation rewrites and CI speedup series also advanced.

## Notable threads

### log: speed up --follow following renames for non-linear history
Justin Tobler benchmarked `git log --stat --follow` on linux.git: 23.5s with the regression commit reverted, 335.9s at the regression, and 38.7s with v5, calling v5 still a significant regression likely to cause server-side file-history timeouts, and suggesting the behavior become opt-in via a new flag with a fresh series. Patrick Steinhardt agreed the cost is too high and proposed first landing a single-patch series introducing a disable knob, offering `--follow=merges`/`log.followMerges`-style opt-in options. Miklos Vajna preferred opt-in over reverting. Junio C Hamano endorsed the opt-in fresh-start suggestion as sensible.

### config: handle NULL values when listing colors and expiry dates
Thomas Bachem posted v3 of the 15-patch series fixing segfaults when settings are given without a value; since v2, `remote.*.tagOpt` and `color.blame.repeatedLines` now warn about and ignore a missing value instead of rejecting it. Phillip Wood remarked that a missing value has long been treated as boolean true, Ben Knoble mused about a future strict-parsing directive, and Junio C Hamano cited the 2005 config parser to agree it is too late to change.

### builtin/rerere: convert to use subcommands

*Long thread (root 2026/10/27/10-46-43): 21 replies so far from 4 participants over 16 days; 2 messages that day.*

Patrick Steinhardt posted v4 of the 12-patch series converting `git rerere` to proper subcommands with `OPT_SUBCOMMAND()` and usage strings. He explained that `RERERE_AUTOUPDATE` takes precedence over `RERERE_NOAUTOUPDATE` in `rerere_cache_setup()`, and v4 now dies via `BUG()` when both flags are passed.

### doc: checkout: rewrite detached HEAD state explanation

*Long thread (root 2026/10/07/14-52-01): 27 replies so far from 4 participants over 36 days; 3 messages that day.*

Julia Evans posted v5 of her rewrite of the detached HEAD explanation as a standalone `gitdetached-head` guide referenced from the detach advice. Junio C Hamano found "Internally in Git, `HEAD` can either be stored as" awkward, suggesting "HEAD can store either", and questioned backticks around `git help detached-head` in the advice message, suggesting single quotes.

### doc: git-diff: rewrite EXAMPLES section

*Long thread (root 2026/11/03/15-33-27): 25 replies so far from 6 participants over 9 days; 3 messages that day.*

Julia Evans posted v3 of the `git diff` EXAMPLES rewrite, which drops the `--cached`/`--staged` SYNOPSIS and DESCRIPTION changes and squashes formatting fixes for the links. Junio C Hamano said he will replace the topic with the v3 patch.

## In brief
- Patrick Steinhardt and Karthik Nayak reviewed the series eagerly creating the ODB transaction temporary directory, questioning the error-message wording and the rationale for eager over lazy creation. *Long thread (root 2026/11/10/00-22-47): 9 replies so far from 3 participants over 2 days; 9 messages that day.*
- Anders Kaseorg fixed a `t2401-worktree-prune.sh` substitution that misfires when the trash path contains "w2", replacing `sed "s/w2/w1/"` with a `cp`; Junio C Hamano called the fix sensible.
- Anders Kaseorg flagged the same `sed "s/w2/w1/"` bug in the post-worktree hook prune test.
- kongfankaikai reported `git repack --drop-filtered` aborting fatally when a filtered blob is referenced by the index, proposing that Git skip such blobs with a warning and drop the rest.
- Junio C Hamano noted the orphaned-commit advice test greps for the wrong `git config` form and suggested a looser pattern.
- Patrick Steinhardt posted v3 of the CI speedup series, including switching t5551's many-refs repositories to reftables to cut macOS runtime from 20 minutes to 2.
- Patrick Steinhardt questioned the per-insertion lookup in the expire-each-reflog-once series, suggesting a sort-and-dedupe after the loop, and asked for a test if the differing `--updateref` behavior is a bug.
