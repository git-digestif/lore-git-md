# Git mailing list daily digest for 2026/10/09

## Long threads

### New `gitmergeconflicts(7)` manual page documentation

*Long thread (root 2026/09/24/14-44-15): 53 replies so far from 6 participants over 16 days; 15 messages that day.*

Julia Evans posted v2 of her six-patch series adding a `gitmergeconflicts(7)` manual page that explains merge conflicts with examples and linking it from git-merge, git-rebase, git-revert, git-cherry-pick, and git-pull instead of re-explaining the process in each command. The v2 changelog lists fixes including removing the SYNOPSIS, a typo fix in git-revert, commit-message prefix normalization, squashing a linter-fix commit, and adding Reviewed-by trailers. Junio C Hamano objected that the Reviewed-by trailers for D. Ben Knoble and Patrick Steinhardt on patch 1/6 were added by the author, since a Reviewed-by can only be offered by the reviewer; Evans acknowledged the point and said she would use `Helped-by` instead. Junio reviewed all six patches, giving detailed suggestions on patch 1/6 (co-author sign-off, semantic conflicts, the `|||||||` marker, aligned fruit examples, "configuration variable" wording, treating `zdiff3` as a failed experiment, clarifying AUTO_MERGE) and smaller notes on the rest, saying he would add his Reviewed-by to patches 2/6, 3/6, and 6/6 as-is and to 4/6 and 5/6 if his suggestions were taken. Evans agreed to several patch 1/6 changes and to dropping the "not obvious how to apply a change" paragraph in git-cherry-pick and the same language in git-revert.

### Fetch refmap inference for shallow-repo remotes

*Long thread (root 2026/09/19/14-47-26): 28 replies so far from 4 participants over 21 days; 2 messages that day.*

Harald Nordgren's v7 series "fetch: avoid fetching every branch of a new remote in a shallow repo" adds `remote.<name>.refmap` as a per-remote default for `--refmap`, infers branches to fetch from local upstreams when only a refmap is configured, adds `git remote add --limited-fetch`, and defaults to limited fetch in shallow repositories. Earlier, Junio C Hamano had reported that merging v7 to `seen` breaks t5505 and t5586; today Nordgren identified the failures as a conflict with 'ch/fetch-followremotehead-lazy-validation' that he expects to disappear in v8. Nordgren also agreed to Junio's earlier review points: renaming the `inferred_branches` variable to `infer_from_refmap`, improving `tracks_this_remote` handling, unifying `add_if_tracking_remote` with the current-branch upstream-fetch code via a proposed `collect_upstream_from_remote()` helper in remote.c, passing a `struct repository *` to `branches_tracking_remote` instead of hardcoding `the_repository`, and fixing stray patch noise.

### Rerere waiting on `MERGE_RR.lock`

*Long thread (root 2026/09/02/08-31-37): 25 replies so far from 4 participants over 38 days; 2 messages that day.*

Thomas Bachem's v6 series makes rerere wait up to `rerere.lockTimeout` for `MERGE_RR.lock`, adds a hidden `--skip-locked` option to `git rerere gc` used by auto maintenance, and lets commands that stop at a conflict warn and go on without rerere when the lock stays busy. Today Patrick Steinhardt reviewed patch 2/3 with replacement commit-message wording and nits (a layering-violation comment, a debatable error return under RERERE_NOWAIT, a proposed dedicated `enum rerere_gc_flags`), and on patch 3/3 said he is not convinced the commit is necessary, proposing to drop it for now while keeping the option to reintroduce it later if users hit the issue in the wild.

### `branch --delete-merged` detecting squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 22 replies so far from 4 participants over 11 days; 7 messages that day.*

Harald Nordgren's patch makes `git branch --delete-merged` treat squash- or rebase-merged branches as merged when an upstream commit since the fork point contains all their changes, printing "Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)". Today Phillip Wood corrected his earlier misremembering: GitHub "Squash and merge" and `merge --squash` create no second parent, so patch ids rather than trees must be used; he hopes detection is enabled by default with a flag to turn it off and says partial clones should fetch the needed blobs with a documentation warning that it can be slow. Kristoffer Haugsbakk said a solid overnight delete operation is worth any slowness, and Nordgren proposed a `full-clean` command bundling delete-merged and prune operations, and asked whether the blobless partial-clone case should be handled now or left as a follow-up.

### Reference copy and rename through transactions

*Long thread (root 2026/09/19/13-33-44): 20 replies so far from 5 participants over 21 days; 2 messages that day.*

Maciej Ciemborowicz's v4 series routes reference copy and rename through ordinary transactions so the reference-transaction hook can observe and reject the complete operation, fixing the v3 bug where a packed-refs preparation error was overwritten by a later success. Today's discussion turned to process: Patrick Steinhardt advised that small, easily reasoned changes are worth submitting even with AI assistance but large complex ones should be avoided, and Karthik Nayak stated he stopped reviewing the author's other patches because his reviews were met with N+1 versions of the series rather than engagement.

## The day in brief

Julia Evans' `gitmergeconflicts` documentation series advanced through a detailed six-patch review, and her `git status` merge-advice patch was approved, with Junio C Hamano saying he will mark it for `next`. Harald Nordgren's fetch refmap series and squash-merge deletion detection continued iterating on review feedback, while the refs transaction series drew discussion about AI-assisted contributions.

## Notable threads

### `status`: suggest `git merge --continue` during merges

*Long thread (root 2026/10/06/13-02-42): 8 replies so far from 4 participants over 4 days; 8 messages that day.*

Julia Evans posted v3 of her patch changing `wt-status.c` merge-conflict advice from "git commit" to `git merge --continue`, consistent with rebase, revert, and cherry-pick, after v2 failed to actually format the commit reference. Phillip Wood said v3 "looks good to me", and Junio C Hamano said he will apply v3 with Wood's Acked-by and mark the topic for `next`.

### `checkout -m` recreating original conflict labels
Phillip Wood posted v3 of his two-patch series teaching the ort machinery to write conflict labels to `.git/MERGE_LABELS` when a merge result contains conflicts, so `git checkout -m <path>` recreates conflicts with the original labels. Junio C Hamano reviewed it as "nicely done" and said "Will replace."

### Repack losing objects to a mid-run ".keep"
Qin ShiCheng's five-patch series fixes `git repack -d` deleting a pack whose objects were never copied when a concurrent push installs a `.keep` mid-run, by handing pack-objects the snapshot repack took at startup. Today Qin explained that a crammed function in the PR description was a GitGitGadget reflow artifact and agreed to hold off until ps/odb-files-alternates settles before sending v4 on top of master.

### SubmittingPatches policy on AI assistance
Scott Chacon's RFC proposes allowing responsible AI-assisted contributions with disclosure rather than rejecting anything that looks AI-generated. Patrick Steinhardt argued against loosening the policy, saying Git already accepts AI-generated code and that the bottleneck is reviewer capacity, which AI is tightening; Junio C Hamano called the argument "well articulated" and added nothing more.

### `config --global` reading both home and XDG files
Delilah Ashley Wu posted v3 of her two-patch series making `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` via `do_git_config_sequence()`, fixing the inconsistency where `--global` read only the former; v3 drops the Windows slash patch and replaces a `success_count` helper with a `global_had_success` boolean.

## In brief
- Jon Simons posted a 15-patch series speeding up client-side refspec matching in push, cutting a 100-delete push to a 100k-ref server from ~2.5s/~6.6s to ~0.15s/~0.19s; Kristoffer Haugsbakk objected to the series ordering failing tests before the fixing commits.
- Christian Ramos reported that SSH signing `valid-before` is checked at the signer's own date and a missing `revocationFile` fails open; Patrick Steinhardt agreed documenting the limitation and suggested failing by default on a missing revocation file with a ":(optional)" path prefix as an escape hatch.
- Muhammed Dilshad A posted a patch to `unpack-trees.c` making sparse checkouts refuse to overwrite untracked local files with the ordinary untracked-overwrite error, matching full checkout behavior.
- Karthik Nayak's patch commits fetched references before backfilling tags so shallow fetches advertise "have" lines; Patrick Steinhardt questioned whether tag backfill needs a full negotiated fetch, and Nayak agreed to strengthen the test to verify objects are not re-fetched.
- A patch series ports Sam Reis's sha1dc Rust approach to C, hashing roughly 2.7x faster; Todd Zullinger and Junio C Hamano clarified licensing: the Git package as a whole is GPLv2-only, but original permissive files such as sha1dc keep their MIT license.
- Junio C Hamano reviewed the v2 blame series defaulting to `HEAD:.git-blame-ignore-revs`, questioning the NUL-byte rejection rationale while calling the partial-clone fix in `peel_to_commit_oid()` reasonable, and flagged Helped-by trailers for contributors who had not commented on v1.
- Curtis Allen Smith's series fixes `git status` disagreeing with `git diff` and `git add` under active conversion by not trusting size changes in `ie_match_stat()`; Junio C Hamano criticized the first patch's commit message as overly verbose and questioned its large diffstat.
