# Git mailing list daily digest for 2026/09/05

## Long threads

### GSoC series adding path keys to `git repo info`

*Long thread (root 2026/07/16/01-21-31): 31 replies so far from 5 participants over 52 days; 1 message that day.*

K Jayatheerth's GSoC series extends `git repo info` with keys that output paths of repository components: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`, most supporting absolute and relative suffixes, with documentation and tests in `t/t1900-repo-info.sh`. Earlier revisions renamed `superproject-working-tree` to `superproject-root`, added `path.cdup` after Junio C Hamano argued it complements `path.git-prefix`, and a build dependency on the `repo->prefix` field was flagged and acknowledged. Previously, Junio C Hamano identified that `get_superproject_working_tree()` does not accept a `repo` parameter and instead relies on the process's current working directory, so `git --git-dir=... repo info path.superproject-root.absolute` returns an empty value. Today K Jayatheerth confirmed the function relies on `xgetcwd()` rather than inspecting the target repository, causing explicit repository contexts like `--git-dir` to break, and committed to updating `get_superproject_working_tree()` to take a `repo` parameter and update its callers, with test coverage for `--git-dir` planned for v6.

## The day in brief

The rerere lock contention between background `gc` and an in-progress rebase drew a sharp exchange over whether proceeding without recording a preimage is an acceptable trade. Two separate fixes for `--force-if-includes` consulting the wrong ref or an uninitialized timestamp advanced with review. And the GSoC `git repo info` path-keys series got a confirmed diagnosis of its `path.superproject-root` bug under explicit `--git-dir` use.

## Notable threads

### Keeping a background rerere gc from killing a rebase

Thomas Bachem's patch addresses `git rerere gc` holding `MERGE_RR.lock` long enough that a rebase's next conflict dies with `LOCK_DIE_ON_ERROR`, leaving the index written but the state `git rebase --continue` needs unwritten. Today he conceded that v3's trade -- proceeding without recording the preimage after `rerere.lockTimeout` -- is "the wrong trade": in v4 every caller will wait `rerere.lockTimeout` and then fail as today, with only `git rerere gc` giving up at once, dropping the `RERERE_LOCK_OR_DIE` flag and retracting his earlier claim that merge and commit could go on without rerere. Junio C Hamano argued that proceeding without recording the preimage is a hard-to-accept regression, noted a mid-rebase failure is recoverable, and proposed loudly warning when `git rerere` fails to record a preimage, with instructions to run `git rerere` manually before resolving.

### Fixing `--force-if-includes` consulting the wrong ref

Tyler Cipriani's two-patch series fixes `--force-if-includes` checking the reflog of the local branch named after the destination branch regardless of what is being pushed, which has caused false rejections and can cause data loss when an unrelated branch is force-pushed. Today Ben Knoble reviewed patch 1/2, concluding that failing the data-loss case and requiring a bypass "seems ok" since the includes check exists to remind users to do such a check, confirmed the detached-HEAD rejection matches the behavior he reported earlier and is addressed by patch 2, and suggested as a small nit clarifying that the detached-HEAD behavior is only being preserved, not changed. He also agreed detached-HEAD pushes cannot currently stop being rejected since no suitable reflog exists, so "this may be all we can do for now."

### `--force-if-includes` when the remote-tracking ref has no reflog

Aleksei Sviridkin's patch fixes `is_reachable_in_reflog()` leaving its timestamp variable uninitialized when the remote-tracking ref has no reflog -- as with a files-backend clone -- which can cut the reflog walk at the first entry and wrongly reject a push. Today he posted v3, changing `timestamp_t date;` to `timestamp_t date = 0;` so the walk is not cut off, adding a test case to `t/t5533-push-cas.sh`, rewording the first paragraph with Junio C Hamano's wording, explaining why zero is the fallback rather than "now" or a fixed age, and dropping the Assisted-by trailer, which he confirmed will not appear again.

### Migrating shell tests to semantic functions

Mark C. Chu-Carroll's two-patch series updates tests to use semantic helpers like `test_path_is_file` instead of `test -f` across 61 files. Today Junio C Hamano objected to converting `test -f` to `test_path_is_file` inside `if` conditionals, explaining that semantic helpers are designed to shout when a test fails whereas a conditional does not want to report system breakage when its condition is merely unsatisfied; after finding many more wrong than good conversions in a handful of hunks, he declined to review the remainder, stating such conversions require understanding what each `test -X` invocation is really checking.

### Participating in Outreachy's December 2026 cohort

Christian Couder's call for the Git community to join Outreachy's December 2026 cohort drew volunteers and plans. Today he thanked Usman, Kaartic and Pablo for volunteering to co-mentor, said that with no objections he will submit a Community Application for Git very soon, and noted mentors still need to sign up and projects must be submitted before the September 11, 2026, 4 pm UTC deadline. He proposed two intern projects -- continuing to remove global state and perhaps libify parts of the code base, and improving how command arguments and options are scanned and parsed by porting hand-rolled code to the parse-options API or his early-scan API -- and asked for opinions and other ideas, while saying he will start poking GitLab about sponsoring an intern again.

## In brief

- Junio C Hamano acknowledged he had misread the pathspec prefix loop in Yannik Tausch's series, which assumes all pathspec elements share the prefix found so far and blindly uses the first element.
- Aleksei Sviridkin posted v3 of the cherry-pick `--no-commit` documentation series, restoring a `test_ref_missing CHERRY_PICK_HEAD` check in the existing conflicting-pick test and dropping the `git commit -c` advice as belonging in git-commit documentation.
- Jeff King proposed bumping the debian-11 CI job to debian-12 in `.github/workflows/main.yml` and `.gitlab-ci.yml` after Debian 11 exited LTS; Junio C Hamano said he had also noticed the job failing and will queue the patch.
- Kristoffer Haugsbakk posted v3 of the series advertising `gitdatamodel(7)` across the documentation, listing it as a concept guide in `git(1)`, linking it from the glossary terms, and adding a See Also link back from the data model page.
- Thomas Bachem agreed to make the auto-maintenance config parameter a `char *` freed by `replay_ctx_release()` in the next version of his sequencer series.
- In the "What's cooking" thread, Kristoffer Haugsbakk asked that `kh/doc-datamodel` stay out of `next` while he prepares a new version addressing Julia Evans's review, and Junio C Hamano asked whether a reroll of `dk/use-nsec-runtime` should be expected.
- A reported typo in the `git-merge` man page example about `ORIG_HEAD` was noted by Kristoffer Haugsbakk to have been fixed in Git 2.51.0 in commit 953049ee.
