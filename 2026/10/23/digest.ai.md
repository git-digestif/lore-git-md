# Git mailing list daily digest for 2026/10/23

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread (root 2026/09/24/14-44-15): 69 replies so far from 6 participants over 30 days; 6 messages that day.*

Julia Evans posted v4 of her six-patch series that adds a new `gitmergeconflicts` manual page explaining merge conflicts with examples, and links to it from the git-merge, git-rebase, git-revert, git-cherry-pick, and git-pull documentation instead of repeating the explanation in each command. The v4 changelog folds in Junio C Hamano's diff3 example, wording around "multi-step operation like a rebase", a "see git-config for more" note for zdiff3, and cleanups such as sentence periods and a more accurate cherry-pick commit message. Ben Knoble said he is happy with v4 and suspects the thread is reaching diminishing returns. Junio C Hamano flagged a tab-indentation inconsistency in the diff3 example and "each removed one fruit to the list" needing "from", briefly reported a `make check-docs` SYNOPSIS lint failure before retracting it, and proposed rewriting the resolution rationale more tightly; otherwise he saw nothing questionable in the six patches.

### `git branch --delete-merged` and squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 47 replies so far from 6 participants over 25 days; 1 message that day.*

Harald Nordgren's series would let `git branch --delete-merged` treat a branch as merged when an upstream commit contains all of its changes, naming that commit in the output. Earlier discussion had converged on a subset-superset ladder of detection levels, but today Harald Nordgren conceded that the `--delete-merged=squashed` proposal (and likely `--delete-integrated=<level>`) is stopped, because `--delete-merged`'s value slot is already required for the upstream `<pattern>` and cannot also carry a level.

### Incremental connectivity check in `connected.c`

*Long thread (root 2026/09/14/09-47-54): 35 replies so far from 3 participants over 40 days; 3 messages that day.*

Kristofer Karlsson posted v5 of the six-patch series adding an incremental mode for the connectivity check, gated behind `transfer.connectivityCheck=incremental` or `"auto"`. The series no longer touches `repo->fetch_if_missing`; instead it reads blobs and trees with `odb_read_object_info_extended()` and `OBJECT_INFO_SKIP_FETCH_OBJECT` so the verifier never lazily fetches, and a corrupt tree is now reported as an error instead of dying. Junio C Hamano affirmed the approach, suggested keeping the reasoning in the commit for future readers, reviewed v5 as looking good, and indicated he will replace the previous version.

### Fetching only tracked branches from a new remote in shallow repos

*Long thread (root 2026/09/19/14-47-26): 32 replies so far from 4 participants over 35 days; 1 message that day.*

Harald Nordgren's v10 series makes `git fetch` avoid fetching every branch of a new remote in a shallow repository: it adds a per-remote `remote.<name>.refmap` config, a `--limited-fetch` option to `git remote add`, and defaults to limited fetch when adding a remote to an already shallow repo. Junio C Hamano said the series appears fairly well done but asked for another person to give it one more read and Ack, noting he was practically the only reviewer of the later iterations and, not being a target audience, might be missing something large.

### New beginner Git tutorial

*Long thread (root 2026/10/06/19-37-00): 22 replies so far from 5 participants over 18 days; 2 messages that day.*

Julia Evans posted v3 of her two-patch series replacing `gittutorial` with a new beginner tutorial covering `git init` through `git push` using only commands available in Git 2.23.0, explicitly not covering branches or collaboration. The v3 changelog notes Git Bash guidance for Windows, a warning about vi as the default editor, a new section on the importance of `git status`, and an explanation of `--set-upstream`. Ben Knoble said the v3 range-diff looks fine with a few small nits he does not want to hold things up over, including dropping "extremely" from the vi description and recommending `:cquit!` alongside `:q!`.

## The day in brief
Several documentation series advanced: Julia Evans posted v4 of the `gitmergeconflicts` man page series and v3 of the new beginner tutorial. Kristofer Karlsson's incremental connectivity check reached v5 with a cleaner no-lazy-fetch design that Junio C Hamano endorsed. On behavior changes, Harald Nordgren conceded that a `--delete-merged=<level>` option is stopped because the option's value slot is already taken, and Junio C Hamano argued against making `@{push}` comparison the default in `git status`.

## Notable threads

### `git status` holding `index.lock`
Mihai Cosma's two-patch series makes `git status` hold `index.lock` only to write it, and fixes a skipHash bug where status could silently revert a concurrent `git add`. Junio C Hamano suggested that Jun Wu and Mihai Cosma work together on a single consolidated series; Jun Wu replied that his "Wait for the index lock instead of failing" series and Mihai's are relatively independent--his changes non-status commands while Mihai's mainly changes status--and preferred keeping them separate, noting the only textual conflict is one function name.

### `git apply` failure on pure rename followed by edit
Hannes Brandstätter-Müller reported that with `core.fileMode=false`, applying a patch series containing a pure rename followed by an edit to the renamed file fails with "error: renamed: wrong type", reproduced on recent versions but not v2.44.0. He diagnosed that in `check_preimage()` a pure rename leaves `previous->new_mode` as zero, causing the later edit to fail the file-type comparison, and attached a tentative fix with a regression test that falls back to `patch->old_mode`, asking for review of both diagnosis and change.

### Default `@{upstream} @{push}` comparison in `git status`

Junio C Hamano argued against the earlier proposal to default `status.compareBranches` to `@{upstream} @{push}`, saying no convincing argument shows people like the option and it would surprise existing users. Harald Nordgren countered that most users have their push branch as upstream due to `git push -u`, limiting their view of triangular workflows, and Junio suggested that whatever encourages `-u` could also suggest setting the configuration.

### Object storage extension in the ODB
Patrick Steinhardt posted v3 of the seven-patch "odb: introduce object storage extension" series, which adds `extensions.objectStorage` with the same syntax as the ref storage extension. v3 splits the object storage URI into an explicit `enum object_storage_format` plus a separate payload, adds `--object-storage-format=` to git-init and git-clone, and propagates the format (without payload) through git-submodule add/update and recursive clone.

### Advertising fetched refs when backfilling tags
Karthik Nayak's v4 patch fixes shallow fetches not advertising newly fetched refs as haves during tag backfill by adding the tips to the transport's `negotiation_include_tips`. Patrick Steinhardt reviewed it as "much saner" but asked about advertising millions of haves and proposed simplifications; Karthik Nayak explained that only OIDs are negotiated so duplicates are removed, and later traced a leak in t5509-fetch-push-namespaces to remote-helper transports that support smart options but do not clean up correctly, an existing bug he will address in a precursor commit.

## In brief
- Patrick Steinhardt reviewed Igor Wiedler's v3 series fixing the connectivity check with alternates in deepening fetches, calling the change to ignore shallow and grafted alternate refs "looks good" and saying no further reroll was needed.
- Patrick Steinhardt posted v2 of "odb/files: reload alternates on retry again", now loading alternates from the primary object directory even when a quarantine directory is active.
- Phillip Wood posted v4 of the seven-patch series reducing redundant index loads and refreshes in `git stash create`/`git stash push`, cutting lstat() calls per changed path.
- Tamir Duberstein posted v3 of "ls-files: reuse and update the untracked cache", releasing the index lock before printing paths and skipping the optional index write with `--debug`.
- Discussion continued on Phillip Wood's proposed community voting process: Junio C Hamano recommended five commits as an easier eligibility bar than two topics in released versions, and Kristoffer Haugsbakk raised the GenAI topic as an example where non-contributors care.
- Patrick Steinhardt reviewed Justin Tobler's tmp-objdir patch merging the "loose-object-idx" file during migration, calling the approach sensible but objecting to the indefinite lock timeout and append-under-lock pattern; Justin Tobler responded point by point.
- Junio C Hamano objected to Taylor Blau's skipping-negotiator patch copying an existing mistake of queueing unparsed parents, proposing a hybrid approach that documents the deliberate difference as #leftoverbits.
