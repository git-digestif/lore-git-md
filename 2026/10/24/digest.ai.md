# Git mailing list daily digest for 2026/10/24

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 39 replies so far from 4 participants over 36 days; 7 messages that day.*

Harald Nordgren's five-patch v10 series changes fetch and remote configuration so that a shallow repository does not negotiate history for every branch of a newly added remote. Patch 1/5 adds a per-remote `remote.<name>.refmap` config that supplies the default `--refmap` value; 2/5 extracts a `collect_upstream_from_remote()` helper in `remote.c`; 3/5 makes a refspec-less `git fetch <name>` infer what to fetch from local branches whose `@{upstream}` is on that remote; 4/5 adds `git remote add --limited-fetch`; and 5/5 defaults to limited fetch when adding a remote to an already shallow repository. Earlier, Junio C Hamano had called the series fairly well done but asked for another reviewer. Today D. Ben Knoble supplied that read: he questioned the `remote_find_tracking()` fallback to `remote->refmap` in 1/5 (asking whether mapped refs outside the normal fetch refspec would be missed, unresolved), asked for clearer commit-message wording and a strvec instead of a string_list in 2/5 and 3/5, judged 4/5 "looks reasonable", and called 5/5 "short and sweet". Harald Nordgren agreed with the `remote_find_tracking()` point and proposed changing it to return the refmap match when `remote->fetch` is empty or already matches, and 0 when `remote->fetch` exists but does not match.

### advice: use global config for default branch name

*Long thread (root 2027/08/29/00-49-58): 27 replies so far from 5 participants over 46 days; 1 message that day.*

Vsevolod Myalitsin's series changes the `defaultBranchName` advice in `advice.c` so that the suggested `git config set advice.defaultBranchName false` uses the global scope, since the advice appears during `git init` and a local-scope setting only affects the repository where the advice was already shown. Earlier iterations debated the scoping mechanism, and Junio C Hamano had posted a simpler v5 that just adds `--global` to the squelch hint. Today David Timber pointed out that the quoted message in the thread carries a bogus future timestamp (dated 2027), warning that future timestamping is a technique spammers use and a "big no no" with modern spam filters, and advised checking the system.

## The day in brief

A regression report landed for v2.56.0: filtered `--unshallow` fetches of a moved ref abort in pack-objects. On the fetch side, D. Ben Knoble completed the requested second review of Harald Nordgren's v10 limited-fetch series, and Taylor Blau conceded a related default-negotiator bug that Junio C Hamano decided should be fixed in the same series.

## Notable threads

### filtered --unshallow of a moved ref aborts in pack-objects

Daniel Zoghalchali reported that since v2.56.0, a fetch combining a tree filter with `--unshallow` (or `--deepen`) aborts in `builtin/pack-objects.c` with "BUG: should_include_obj should only be called on existing objects" when the fetched ref has moved past the local tip in a non-partial shallow clone, whereas v2.55.0 completes the same fetch. He traced the regression to d43b9acb05 ("setup: unify setup of shallow file"), which exports `GIT_SHALLOW_FILE` so the pack-objects grandchild inherits the empty shallow file, stops treating the local tip as a shallow boundary, and walks to a promisor commit whose root tree was never fetched. He confirmed the mechanism by wrapping the repack child: unsetting `GIT_SHALLOW_FILE` makes 2.56.0 pass, while setting it empty makes 2.55.0 fail the same way. He also noted the BUG is reachable on 2.55.0 without the environment change, via a filtered `--unshallow` of the local tip followed by a filtered fetch of a moved ref.

### negotiator/skipping.c: parse parents before queueing

Taylor Blau's patch fixes the skipping fetch negotiator in `negotiator/skipping.c`, whose date-sorted queue could hold unparsed parents with a zero date, letting old ref tips sort ahead of a recent common ancestor. Today Taylor Blau conceded that Junio C Hamano is right that the default negotiator has a related bug (marking commits SEEN before parsing succeeds), accepted a hybrid approach, and offered to fix the default negotiator alongside this series. Junio C Hamano decided that, given a clear grasp of the existing bug and a straightforward fix, the default negotiator fix should be done as part of this series.

### reflog: add --operations and --historical-branch

Harald Nordgren's v2 series adds `--operations` (one entry per command, folding rebases) and `--historical-branch` (decorating each entry with the branch HEAD was on) to `git reflog` and `git log -g`. Today Phillip Wood was skeptical of `--operations` in its current form, since not all working-tree-modifying commands update the reflog and multi-commit commands like `git cherry-pick A B C` are indistinguishable from repeated invocations; he liked `--historical-branch` but would prefer a `%gb` format specifier, and noted accurate branch names require handling `git rebase <upstream> <branch>`. D. Ben Knoble agreed the series is not a true operations log and suggested a structured event log might be simpler. Junio C Hamano liked making default single-branch reflog output easier to read, and Harald Nordgren agreed that implementing `%gb` as a format specifier would radically change the series and drop `--operations` altogether.

### branch: delete gone-upstream branches only when proven merged

The RFC asks how to close the gap where `git branch --delete-merged` never deletes a topic branch whose upstream was deleted by the server after a merge. Today DonQuinleone reported testing against Git 2.56.0: with `push.default=current` the dry run offers to delete the topic after the PR merges with no code change, so the gap only affects users who run `git push -u`; he confirmed `git branch --list --merged=origin/master | xargs git branch -d` fails because `-d` re-checks each branch against its gone upstream; and he proposed `git branch -d --upstream-gone --merged=origin/master`, where the explicit `--merged` commit serves as both selection filter and deletion proof, asking whether "selection is not proof" is an acceptable basis. D. Ben Knoble defended the composable `--delete-merged=into:origin/master` syntax and explained `--into` is needed because once the upstream is gone there is nothing to check against.

### Documentation: propose community voting process

Discussion of the proposed community voting process continued from the Contributor's Summit notes. Tuomas Ahola noted that Peff's proposed summit threshold of 50 merged patches was substantially higher than the thresholds in the current voting-process thread. Weijie Yuan said he feared he was not qualified enough to justify an equal say in Git's direction, and later apologized for misreading the exchange. Junio C Hamano said he did not know whether 50 or 200 merged patches was the more appropriate threshold but considered both more realistic, and clarified that his earlier "2 vs 5" comparison only showed commit count exceeds topic count.

## In brief

- D. Ben Knoble still supports making `@{upstream} @{push}` the default for `status.compareBranches` but agreed with Junio C Hamano that a stronger case is needed; Junio C Hamano replied that he strongly assumes users only push new commits to `@{push}`.
- Srujith A Poojary posted v4 of the `diff-delta.c` patch using `st_mult`/`st_add`/`st_add3` for safe allocation arithmetic on 32-bit platforms, with a step-by-step commit-message analysis after Junio C Hamano's review of v3.
- Aniket Gupta posted v2 using `Q_()` in `object-name.c` so the reflog entry-count message uses the singular for one entry, updating `t/t1506` to match "entr" and adding a test in `t/t1400-update-ref.sh`.
- Jasper Mayone posted v2 of `git worktree list --branch=<branch>`, a repeatable option listing only worktrees with the given branch checked out, dropping the earlier `--merged`/`--no-merged` approach.
- Tanga Joris could not reproduce the `git apply --check` assertion crash with the patch as quoted, but confirmed it triggers when the `--- /dev/null` line precedes `new file mode` in the header.
- Junio C Hamano said the `remote.<name>.followRemoteHEAD` deferral patch has a textual conflict with Harald Nordgren's fetch-refmap work, keeping it out of `seen`.
- Laveesh Bansal submitted a patch making `git reflog expire --dry-run --verbose` say "would prune" instead of "prune" for named refs, with tests in `t/t1410-reflog.sh`.
- Taylor Blau defended pinning GitHub Actions in `.github/workflows/main.yml` to full commit IDs as a security measure, and Junio C Hamano agreed a reworded commit message would be a good way forward.
- Tanga Joris posted a patch dropping `NO_PARSEOPT` from `diff-pairs`, `fast-import`, `patch-id`, and `var` in `git.c` so bash completion can offer their options.
- Sami Jawhar submitted a one-line patch setting a placeholder gitdir in `oss-fuzz/fuzz-commit-graph.c` so the fuzzer passes `prepare_repo_settings()` and reaches the optional commit-graph chunks.
- Shashank S submitted v3 of the `prune-packed` patch removing `USE_THE_REPOSITORY_VARIABLE`, correcting the `UNUSED`-macro rationale per review.
- Aniket Gupta posted a four-patch series moving `core.autocrlf`, `core.eol`, and `core.safecrlf` globals into `struct repo_config_values` as part of the libification effort.
