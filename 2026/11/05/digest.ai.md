# Git mailing list daily digest for 2026/11/05

## Long threads

### Removing gittutorial-2 from the documentation

*Long thread (root 2026/09/28/20-25-19): 32 replies so far from 4 participants over 39 days; 4 messages that day.*

This is a documentation change that deletes `gittutorial-2` and all references to it, leaving a stub that redirects readers to `gitdatamodel` and `gitcore-tutorial`, with the stated goal of removing obsolete tutorial material. Earlier, the series went through v2 and v3, with v3 removing `gittutorial-2` from `command-list.txt` so that `git(1)` would not advertise an obsolete tutorial, and Junio C Hamano asked that changes to `Documentation/gittutorial.adoc` be left out of the series. Today, Tuomas Ahola reported that the removal topic has graduated to `master` and asked Junio C Hamano to pick up the rerolled `ta/command-list-guides-sync-lint` topic; after Junio asked what the `git cherry` output was meant to convey, Tuomas confirmed the patch was the result of the rebasing he had earlier said he would do and that the topic is not in `seen`. Junio then noted that the thread still has an unanswered review comment by Julia Evans and asked whether it is ready to be picked up.

### Shallow-clone hint for `<rev>~N` failures in object-name

*Long thread (root 2026/09/20/09-53-33): 23 replies so far from 6 participants over 47 days; 1 message that day.*

This is a change to the object-name code that adds a hint when a `<rev>~N` walk in a shallow clone fails exactly at a recorded shallow boundary, naming the remote and branch and suggesting the exact `--deepen` needed. Earlier, v3 of the patch dropped the `git log -<n>`/`--since` extension and fixed a double-hint bug for chained `^` resolution, and a discussion about review capacity followed Junio C Hamano's warning that he would discard unreviewed topics from `seen`. Today, Junio C Hamano observed that after a few days nobody in the thread had expressed support for the shallow-clone `<rev>~N` error hint topic, and that the discussion thread meant to invite supporters had instead been cluttered with messages about the review process.

### Quarantine alternate refs in the object database

*Long thread (root 2026/10/28/15-18-24): 21 replies so far from 5 participants over 9 days; 4 messages that day.*

This is a change to the ODB alternate-refs code so that `rev-list --alternate-refs` no longer lists the repository's own refs, including hidden ones, during receive-pack's quarantine connectivity check. Earlier, v2 added a "borrowed" flag recognizing the own object directory; Patrick Steinhardt objected that it was incomplete, while Jeff King argued for fixing the alternate-refs code itself. Today, Kristofer Karlsson posted v3, reverting to the v1 approach of skipping the repository's own object directory in the alternate-refs code with no special quarantine handling, with a perf test showing a push into a repository with ~200,000 hidden refs improving from 0.29s to 0.10s. Jeff King reviewed it and said it looks good, Junio C Hamano said he will replace the previous version, and later marked the topic ready for `next`.

## The day in brief
Kristofer Karlsson's v3 of the quarantine alternate-refs fix was reviewed positively and marked ready for `next`, while the gittutorial-2 removal graduated to `master`. Derrick Stolee retracted his own fsmonitor/untracked-cache patch in favor of Ilia K's, which he reviewed as LGTM. Christoph Weiss reported that geometric maintenance prunes commits without updating the commit-graph, breaking repository health checks.

## Notable threads

### Geometric maintenance prunes commits without updating the commit-graph
This is a bug report against the maintenance code: after updating Forgejo from 16 to 17, which switches Git from 2.52 to 2.54 and changes the default maintenance strategy from `gc` to `geometric`, repository health checks started failing. Christoph Weiss reported that under `geometric`, maintenance prunes commits without updating the commit-graph, and provided a reproduction script.

### fsmonitor untracked-cache invalidation after losing history
This concerns the integration between fsmonitor and the untracked cache, where a failed fsmonitor token request can leave stale valid bits that later `git status` runs trust. Derrick Stolee acknowledged that Ilia K's patch addresses the same issue with the stronger approach of preserving as much of the untracked cache as possible, completed his review, and retracted his own patch, stating that Ilia's should be taken instead.

### fsmonitor trivial response and the untracked cache
This is a fix to the fsmonitor/untracked-cache integration: when the fsmonitor daemon returns a trivial response after losing event history, cached directory valid bits are saved unchanged and later `git status` runs trust stale data. Derrick Stolee reviewed Ilia K's patch, which selectively invalidates only entries whose stat data no longer matches, judged it better than a full-clear approach because it showed no measurable performance impact where the full clear showed a 50-75% penalty on untracked cache tests, and concluded it is LGTM. Junio C Hamano agreed with Stolee's recursion-depth reasoning and said that if there is truly no worktree, invalidating and continuing is probably safer.

### Submodule commit abbreviation using the submodule repository
This is a fix in the diff/submodule code: commit IDs in a submodule diff header belong to the submodule's object database, but `strbuf_add_unique_abbrev()` used the superproject's. The patch uses the submodule repository when available and falls back otherwise. Junio C Hamano reviewed the v6 patch as "very good" and queued it, marking it ready for `next`.

### Replaying 2-parent merges in git replay/history
This is a feature for the `replay` builtin backing `git replay` and `git history`, which currently dies on merge commits in the rewrite path; the series wires Elijah Newren's 2-parent merge replay algorithm into `replay_revisions()`. Philippe Blain reported that when replaying a merge whose first parent was amended, the pre-seeding incorrectly maps the second parent to the onto commit, changing the merge-base and causing the replay to conflict, suggested mentioning the pre-seeding in the commit message, proposed adding the replay design notes as a technical document once the series is out of RFC, and stated the series is already a massive time saver for him.

## In brief
- Tanga Joris tested v3 of moving `pack.packSizeLimit` into `struct repo_config_values` (t1300, t5300 and t7700 pass, the limit works) but noted that patch 2/2 adds a member to a structure that Junio's `jc/freeze-repo-config-values` topic says should not gain new members, and that v3 does not merge cleanly with `seen`.
- Tuomas Ahola posted v2 updating `Documentation/CodingGuidelines` to require Perl 5.26.0 via `require v5.26`; Junio C Hamano replied "Looks good. Will queue."
- Tanga Joris submitted a patch to `builtin/annotate.c` passing arguments to `cmd_blame()` unchanged so `git annotate --<TAB>` offers annotate's own options; Junio C Hamano called it a nice simplification and said it looks good.
- Kristofer Karlsson reported that Linus Torvalds's describe commit-date-skew patch makes an expected-failure test pass only with a commit-graph; Junio C Hamano suggested freezing the test to always run under `GIT_TEST_COMMIT_GRAPH`.
- Jeff King argued that Patrick Steinhardt's loose-object stale-cache fix addresses the observed case but leaves related stale-cache cases, suggesting `QUICK` callers tolerate false negatives and reconsidering the earlier cache-flush change.
- Yopa Nelly posted v2 of `promisor.fetchOrder` with the parsed list per-repository in `struct promisor_remote_config` and a two-pass walk in `try_promisor_remotes()`; Junio C Hamano asked how it meshes with server-side promisor ordering work.
- Azuama Odinaka Maximus posted v4 of the rev-list early option scan fix, using `parse_long_opt()` to skip separate arguments of revision options; Tanga Joris had reported that v3 fixed only `--grep`.
- Ravi Mistry proposed defaulting `http.retryAfter` to 1 second; brian m. carlson objected that 1s is not a reasonable backoff for a 429, suggesting 30s with at least a 10s floor.
- Adrian Ratiu submitted `ls-remote --show-object-format`; Patrick Steinhardt reviewed it as sensible and proposed a prefixed `object-format: %s` output, which Ratiu agreed to adopt in v2.
- Julia Evans posted v2 rewriting the `git-diff` EXAMPLES section; Junio C Hamano objected to leading the SYNOPSIS with `--staged` and said he would add a squashable fix-up changing it to `(--cached|--staged)`.
