# Git mailing list daily digest for 2026/11/11

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 72 replies so far from 5 participants over 54 days; 6 messages that day.*

Harald Nordgren's v13 series separates which refs a remote fetches from where they are stored via `remote.<name>.fetchRefmap` and `--limited-fetch`, so shallow clones of a new remote skip fetching every branch. He asked for hopefully final reviews. Phillip Wood criticized the cover letter for restating only the subject without explaining the problem or trade-offs; Junio C Hamano agreed and was not interested in patch 6's `@{push}` support. D. Ben Knoble asked whether that `@{push}` failure is a bug from patch 3, and Junio agreed making `@{push}` consult `remote.<name>.fetchRefmap` is a logical part of the work.

### repo: add path keys to `git repo info`

*Long thread: 60 replies so far from 6 participants over 119 days; 3 messages that day.*

K Jayatheerth's GSoC series exposes repository paths through `git repo info` keys such as `path.toplevel`, `path.superproject-root`, and `path.index`. Today he agreed to drop the `path.hooks` and `path.cdup` patches after Phillip Wood argued they encourage an outdated interface and duplicate relative paths, and to avoid hyphens in the superproject key after Junio C Hamano noted Git avoids dashes in configuration-style names. Phillip Wood looks forward to v9.

### branch: let `--delete-integrated` find squash-merged branches

*Long thread: 55 replies so far from 6 participants over 44 days; 2 messages that day.*

Harald Nordgren's series adds `git branch --delete-integrated <pattern>`, deleting branches whose changes landed upstream via squash or rebase merges and naming the commit with "landed as". Jasper Mayone reviewed v6: he flagged that `repo_read_index()` runs per changed file and re-walks the index in sparse checkouts, noted merge attributes now come from the worktree's index rather than the upstream commit, and proposed a test for a merge that leaves upstream changed. He concluded the patch looks good to him.

### worktree: treat a branch being rebased or bisected as checked out

*Long thread: 33 replies so far from 8 participants over 22 days; 2 messages that day.*

Jasper Mayone's series makes `git worktree remove` refuse worktrees mid-operation, documents reading `%(worktreepath)` safely, and stops `git branch` from marking a branch with "+" in the worktree rebasing it. He posted v5: patch 3 now uses a map in `branch.c` for one-lookup `branch_checked_out_worktree()`, new patch 4 adds `%(currentworktree)`, and patch 5 uses it in the default format. Ben Knoble said the new patch 1 wording is also OK by him.

### t: add `test-tool genbytes` to avoid slow `tr(1)` on macOS

*Long thread: 30 replies so far from 5 participants over 5 days; 5 messages that day.*

Kristofer Karlsson's patch adds `test-tool genbytes <char> [<count>]` so tests stop piping `genzeros` through macOS's slow `tr(1)`, which burns over 95 seconds of CPU on t4205. Patrick Steinhardt called v3's argument handling great; Junio C Hamano noted a new `genzeros | tr` use in `t/t1600-index.sh` also needs converting; Kristofer posted v4 rebased to catch that callsite.

## The day in brief
Benchmarks showed the `--follow` rename-following change still far slower than before its regression, drawing calls from Patrick Steinhardt and Junio C Hamano for an opt-in toggle. Harald Nordgren's shallow-fetch series reached its v13 review round, and K Jayatheerth trimmed the `git repo info` path keys after review feedback.

## Notable threads

### log: speed up `--follow` following renames for non-linear history
Justin Tobler benchmarked `git log --stat --follow` on linux.git: 23.5s with the change reverted, 335.9s at the regression, 38.7s with v5 -- still a significant regression he expects to cause server-side timeouts, so he proposed an opt-in flag and a fresh series. Patrick Steinhardt agreed the cost is too high and wants a disable knob fast since GitLab deploys from master; Miklos Vajna preferred opt-in over reverting; Junio C Hamano endorsed the opt-in fresh start.

### config: fix segfaults on settings given without a value
Thomas Bachem posted v3 of a 15-patch series fixing segfaults when settings like `rebase.missingCommitsCheck`, color slots, or `remote.*.tagOpt` are given without a value; v3 makes `remote.*.tagOpt` and `color.blame.repeatedLines` warn and ignore a missing value. Phillip Wood, Ben Knoble, and Junio C Hamano discussed the long-standing valueless-means-true behavior, agreeing it is too late to change.

### builtin/rerere: convert to use subcommands

*Long thread: 21 replies so far from 4 participants over 16 days; 2 messages that day.*

Patrick Steinhardt posted v4 of a 12-patch series converting `git rerere`'s custom subcommand parsing into proper subcommands with usage strings; v4 polishes `RERERE_{NO,}AUTOUPDATE` docs and dies via `BUG()` when both flags are passed, since `RERERE_AUTOUPDATE` otherwise silently wins.

## In brief
- Anders Kaseorg fixed `t/t2401-worktree-prune.sh` misfiring when the trash path contains "w2", replacing a `sed` substitution with `cp`; Junio C Hamano called it sensible.
- Patrick Steinhardt and Karthik Nayak reviewed Justin Tobler's series creating the ODB transaction temporary directory eagerly, debating the prefix name and error wording. *Long thread: 9 replies so far from 3 participants over 2 days; 9 messages that day.*
- Junio C Hamano noted a test in the orphaned-commit-check advice series greps for the wrong `git config` form and suggested a looser pattern.
- Patrick Steinhardt assessed that zlib 1.4's raw-deflate change is harmless to Git, since `archive-zip.c` handles `Z_STREAM_END` correctly.
- Patrick Steinhardt posted v3 of CI speedups, including reftables for many-refs tests cutting macOS t5551 from 20 minutes to 2.
- kongfankaikai reported `git repack --drop-filtered` aborts fatally when a droppable blob is index-referenced, asking whether it could skip those instead.
