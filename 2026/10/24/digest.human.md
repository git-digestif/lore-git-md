# Git mailing list daily digest for 2026/10/24

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 39 replies so far from 4 participants over 36 days; 7 messages that day.*

Harald Nordgren's five-patch v10 series changes fetch behavior in shallow repositories so that adding a remote does not commit to fetching every branch. It adds a `remote.<name>.refmap` config as the per-remote default for `--refmap`, extracts a `collect_upstream_from_remote()` helper in `remote.c`, makes a refspec-less `git fetch <name>` infer what to fetch from local branches whose `@{upstream}` is on that remote, adds `--limited-fetch` to `git remote add`, and defaults to it when adding a remote to an already shallow repository. Earlier, Junio C Hamano had called the series fairly well done but asked for another reviewer. Today D. Ben Knoble supplied that read: he questioned whether the `remote_find_tracking()` fallback to `remote->refmap` in 1/5 could miss mapped refs outside the fetch refspec (e.g. for `--delete-merged`), asked for clearer commit-message wording and a strvec instead of a string_list in 2/5 and 3/5, judged 4/5 "looks reasonable", and called 5/5 "short and sweet". Harald Nordgren agreed with the `remote_find_tracking()` point and proposed using the refmap when `remote->fetch` is empty or matches, returning 0 when fetch is present but does not match.

### advice: use global config for default branch name

*Long thread: 27 replies so far from 5 participants over 46 days; 1 message that day.*

Vsevolod Myalitsin's series makes the `defaultBranchName` advice shown during `git init` recommend a global config scope, since a local-scope setting only affects the repository where the advice was already shown while the advice reappears on the next `git init`; it passes the whole `advice_setting` to `vadvise()` and adds a scope hint. Earlier discussion had converged on adding `--global` to the squelch hint. Today David Timber pointed out that a quoted message in the thread carries a bogus future timestamp (dated 2027), warning that future timestamping is a technique spammers use and a "big no no" with modern spam filters, and advised checking the system.

## The day in brief

D. Ben Knoble completed the requested second review of Harald Nordgren's v10 fetch series for shallow repositories, with one agreed follow-up on `remote_find_tracking()`. Daniel Zoghalchali reported a v2.56.0 regression where a filtered `--unshallow` of a moved ref aborts in pack-objects. Taylor Blau conceded a related bug in the default fetch negotiator, and Junio C Hamano decided the fix belongs in the same series.

## Notable threads

### filtered --unshallow of a moved ref aborts in pack-objects

Daniel Zoghalchali reported that since v2.56.0, a fetch combining a tree filter with `--unshallow` (or `--deepen`) aborts in a non-partial shallow clone with "BUG: should_include_obj should only be called on existing objects" when the fetched ref has moved past the local tip, while v2.55.0 completes the same fetch. He traced the regression to d43b9acb05 ("setup: unify setup of shallow file"), which exports `GIT_SHALLOW_FILE` so the pack-objects grandchild inherits the empty shallow file, stops treating the local tip as a shallow boundary, and walks to a promisor commit whose root tree was never fetched. He confirmed the mechanism by unsetting `GIT_SHALLOW_FILE` for the repack child (2.56.0 passes) and setting it empty (2.55.0 fails the same way), and noted the BUG is also reachable on 2.55.0 via a filtered `--unshallow` of the local tip followed by a filtered fetch of a moved ref.

### negotiator/skipping.c: parse parents before queueing

Taylor Blau's patch makes the skipping fetch negotiator parse each unseen parent before pushing it onto the date-sorted queue, so old ref tips no longer sort ahead of unparsed parents with zero dates. Today Taylor conceded that Junio C Hamano is right that the default negotiator has a related bug (marking commits SEEN before parsing succeeds), accepted a hybrid approach, and offered to fix the default negotiator alongside this series with a patch he already has locally. Junio C Hamano decided that, given a clear grasp of the existing bug and a straightforward fix, the default negotiator fix should be done as part of this series.

### reflog: add --operations and --historical-branch

Harald Nordgren's v2 adds `--operations` (one entry per command, folding rebases) and `--historical-branch` (decorating each entry with the branch HEAD was on) to `git reflog` and `git log -g`. Today Phillip Wood was skeptical of `--operations` in its current form, since not all working-tree-modifying commands update the reflog and multi-commit commands like `git cherry-pick A B C` are indistinguishable from repeated invocations; he preferred implementing the historical branch as a `%gb` format specifier and would be happy to change the default HEAD format. D. Ben Knoble agreed the series is not a true operations log and suggested a structured event log might be simpler than reflog parsing. Junio C Hamano liked the direction of a clearer default single-branch reflog, and Harald Nordgren agreed that `%gb` would radically change the series and drop `--operations` altogether.

## In brief

- D. Ben Knoble defended a composable `--delete-merged=into:origin/master` syntax and proposed `git branch --list --merged-upstream=origin/main | xargs git branch -d`; DonQuinleone reported that with `push.default=current` the gone-upstream gap only affects `git push -u` users and proposed `git branch -d --upstream-gone --merged=origin/master`.
- Tanga Joris could not reproduce the `git apply --check` assertion crash with the patch as quoted but confirmed it triggers when `--- /dev/null` precedes `new file mode` in the header.
- Srujith Poojary posted v4 of the `diff-delta.c` safe-integer-arithmetic patch with a step-by-step 32-bit overflow analysis after Junio C Hamano's review of v3.
- Junio C Hamano noted the deferred `remote.<name>.followRemoteHEAD` validation patch has a textual conflict with Harald Nordgren's fetch-refmap work and suggested it could wait until that topic reaches `next` and then be rebased.
- D. Ben Knoble supported making `@{upstream} @{push}` the default in `git status` but agreed with Junio C Hamano that a stronger case is needed; Junio replied that he strongly assumes users only push new commits to `@{push}`.
- Jasper Mayone posted v2 of `git worktree list --branch=<branch>`, dropping the earlier `--merged` reachability code in favor of the repeatable branch selector.
- Taylor Blau defended pinning GitHub Actions to full commit IDs as a security measure, citing the burden on corporate forks; Junio C Hamano agreed a reworded commit message would be a good way forward.
