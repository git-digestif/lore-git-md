# Git mailing list daily digest for 2026/10/09

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 53 replies so far from 6 participants over 16 days; 15 messages that day.*

Julia Evans posted v2 of her six-patch series adding a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` instead of repeating the guidance in each command. Earlier rounds had drafted a "WHAT IS A MERGE CONFLICT?" section and revised the "ours"/"theirs" wording after review from Junio C Hamano and D. Ben Knoble. Today Junio objected that the `Reviewed-by:` trailers on patch 1/6 were added by the author rather than offered by the reviewers themselves; Julia agreed to use `Helped-by:` instead. Junio reviewed each patch, suggesting changes to the new page (co-author sign-off, explaining the `|||||||` marker, treating `zdiff3` as a failed experiment) and simplifications in the command docs, and said he would add his `Reviewed-by:` to patches 2/6, 3/6, and 6/6 as-is and to 4/6 and 5/6 if his suggestions were taken.

### Fetch refmap inference for shallow repositories

*Long thread: 28 replies so far from 4 participants over 21 days; 2 messages that day.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository by adding `remote.<name>.refmap`, inferring branches to fetch from local upstreams when only a refmap is configured, and defaulting to limited fetch in shallow clones. After v7 fixed a fatal `--refmap` error, Junio had asked for wording, variable-naming, and code-structure improvements and reported test failures when the series is merged to `seen`. Today Harald identified those t5505/t5586 failures as an interaction with the `ch/fetch-followremotehead-lazy-validation` topic that he expects to disappear in v8, and agreed to rename `inferred_branches` to `infer_from_refmap`, unify the upstream-fetch code paths via a proposed `collect_upstream_from_remote()` helper in `remote.c`, pass a `struct repository *` instead of hardcoding `the_repository`, and check whether the current branch's upstream could be added twice.

### Rerere waiting on `MERGE_RR.lock`

*Long thread: 25 replies so far from 4 participants over 38 days; 2 messages that day.*

Thomas Bachem's v6 series makes rerere wait up to `rerere.lockTimeout` for `MERGE_RR.lock`, adds a hidden `gc --skip-locked` for auto maintenance, and lets commands that stop at a conflict continue without rerere when the lock stays busy. Today Patrick Steinhardt reviewed patches 2/3: he supplied replacement commit-message wording for the `--skip-locked` patch, called a new comment in `rerere.c` a layering violation, suggested a dedicated `enum rerere_gc_flags` for `rerere_gc()`, and said he is not convinced patch 3 is necessary, proposing to drop it while keeping the option to reintroduce it if users hit the issue.

### `--delete-merged` detecting squash-merged branches

*Long thread: 22 replies so far from 4 participants over 11 days; 7 messages that day.*

Harald Nordgren's patch makes `git branch --delete-merged` treat a branch as merged when an upstream commit since the fork point contains all its changes, printing where the work landed. Earlier review from Phillip Wood and D. Ben Knoble focused on efficiency: per-branch upstream walks are costly, and patch-id comparison is needed because squash merges create single-parent commits that are not tree-same to the topic tip. Today Phillip corrected his earlier tree-based assumption and agreed patch ids must be used, endorsing detection on by default with a flag to turn it off and documentation warning that partial clones may be slow. Kristoffer Haugsbakk said he would run the cleanup overnight and does not mind the cost; Harald asked whether the blobless partial-clone case should be handled now or as a follow-up.

### Reference copy and rename through transactions

*Long thread: 20 replies so far from 5 participants over 21 days; 2 messages that day.*

Maciej Ciemborowicz's v4 series routes reference copy and rename through ordinary transactions so the reference-transaction hook sees the whole operation, fixing the v3 regression in t0600/t5510 where a packed-refs preparation error was overwritten. Today Patrick Steinhardt advised that small AI-assisted changes are worth submitting but large complex ones should be avoided, and Karthik Nayak said he stopped reviewing the author's other patches because reviews were met with new versions rather than engagement.

## The day in brief
Julia Evans' `gitmergeconflicts(7)` documentation series reached v2 with detailed review from Junio C Hamano, and her `git status` patch suggesting `git merge --continue` was approved for `next`. Harald Nordgren's fetch-refmap series and squash-merge detection for `--delete-merged` both absorbed reviewer feedback on structure and performance.

## Notable threads

### `checkout -m` recreating original conflict labels
Phillip Wood posted v3 of his two-patch series teaching the ort machinery to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` recreates conflicts with the original labels. Junio C Hamano reviewed it as "nicely done" and said "Will replace."

### `git status` suggesting `git merge --continue`

*Long thread: 8 replies so far from 4 participants over 4 days; 8 messages that day.*

Julia Evans posted v3 of her patch changing the merge-conflict advice in `wt-status.c` from `git commit` to `git merge --continue`, consistent with rebase, revert, and cherry-pick. Phillip Wood said the version looks good, and Junio C Hamano said he will apply it with Phillip's `Acked-by:` and mark the topic for `next`.

### SSH signing trust limits
Christian Ramos reported that `valid-before` is checked against a date the signer writes and that a missing `gpg.ssh.revocationFile` fails open, so an SSH signing key cannot be reliably stopped from being trusted. Patrick Steinhardt argued there is no proper fix for the timestamp in a distributed system but agreed documentation should avoid a false sense of trustworthiness, and suggested failing by default on a missing revocation file with a `:(optional)` path prefix as an escape hatch; Phillip Wood endorsed both points and welcomed a test.

## In brief
- **Patrick Steinhardt on AI review bottleneck**: Patrick Steinhardt argued the review bottleneck is tightening as AI-generated slop increases, warning that loosening the AI policy without solutions would make the problem significantly worse; Junio C Hamano replied "Well articulated."

- Qin ShiCheng's series fixing `git repack -d` losing objects to a mid-run `.keep` will be held until `ps/odb-files-alternates` settles before a v4.
- Delilah Ashley Wu posted v3 making `git config --global` read both `$HOME/.gitconfig` and the XDG config file.
- Muhammed Dilshad A posted a patch making sparse checkout refuse to overwrite untracked files like ordinary checkout.
- Jon Simons posted a 15-patch series speeding up client-side push refspec matching; Kristoffer Haugsbakk objected to separating failing tests from their fixes.
- Karthik Nayak's patch commits fetched references before backfilling tags so shallow fetches advertise `have` lines; Patrick Steinhardt questioned the test's coverage.
