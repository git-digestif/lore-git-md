# Git mailing list daily digest for 2026/10/23

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 69 replies so far from 6 participants over 30 days; 6 messages that day.*

Julia Evans posted v4 of her six-patch series adding a `gitmergeconflicts` man page that explains merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation instead of repeating the explanation in each command. The v4 changelog folds in Junio C Hamano's diff3 example and cleanups such as sentence periods and a more accurate cherry-pick commit message. Ben Knoble said he is happy with v4 and suspects the thread is reaching diminishing returns. Junio C Hamano flagged a tab-indentation inconsistency in the diff3 example and the phrase "each removed one fruit to the list" needing "from", retracted an earlier `SYNOPSIS` lint concern, and otherwise found nothing questionable in the six patches.

### `git branch --delete-merged` and squash-merged branches

*Long thread: 47 replies so far from 6 participants over 25 days; 1 message that day.*

Harald Nordgren's series would let `git branch --delete-merged` treat a branch as merged when an upstream commit since the fork point has the branch's version of every file it changed, naming that commit in the output as "landed as ...". After Junio C Hamano objected to loosening the default and the thread explored `--delete-integrated` and level names, Harald Nordgren conceded today that the `--delete-merged=squashed` proposal is stopped: the option's value slot is already required for the upstream `<pattern>` and cannot also carry a level.

### Incremental connectivity check in `connected.c`

*Long thread: 35 replies so far from 3 participants over 40 days; 3 messages that day.*

Kristofer Karlsson posted v5 of the six-patch series adding an incremental mode to the connectivity check, gated behind `transfer.connectivityCheck=incremental` or `auto`. The series no longer touches `repo->fetch_if_missing`; it reads blobs and trees with `odb_read_object_info_extended()` and `OBJECT_INFO_SKIP_FETCH_OBJECT` so the verifier never lazily fetches, and reports a corrupt tree as an error instead of dying. Junio C Hamano reviewed v5, said the incremental changes look good, and indicated he will replace the previous version.

### Fetching only tracked branches from a new remote in a shallow repo

*Long thread: 32 replies so far from 4 participants over 35 days; 1 message that day.*

Junio C Hamano said Harald Nordgren's v10 series -- which avoids fetching every branch of a new remote in a shallow repository via `remote.<name>.refmap`, a `--limited-fetch` option to `git remote add`, and a shallow-repo default for it -- appears fairly well done, but asked another person to give it one more read and Ack, noting he was practically the only reviewer of the later iterations and might be missing something as a non-target user.

### New beginner `gittutorial` page

*Long thread: 22 replies so far from 5 participants over 18 days; 2 messages that day.*

Julia Evans posted v3 of her two-patch series replacing the old `gittutorial` with a new 782-line beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0 and explicitly not covering branches or collaboration. Ben Knoble called the v3 range-diff fine with a few small nits: drop "extremely" from the `vi` description, say "bottom-left" for the tilde characters, and ideally recommend `:cquit!` alongside `:q!`.

## The day in brief
Julia Evans advanced two documentation series, posting v4 of the `gitmergeconflicts` man page and v3 of the new beginner tutorial. Kristofer Karlsson posted v5 of the incremental connectivity check after removing its dependence on `repo->fetch_if_missing`, and Junio C Hamano judged the incremental changes good. Harald Nordgren conceded that the `--delete-merged=squashed` option design is stopped because the option's value slot is already taken by the upstream pattern.

## Notable threads

### `git status` comparing against both upstream and push branch by default
Harald Nordgren proposed making `@{upstream} @{push}` the default for `status.compareBranches` so `git status` shows both comparisons. Junio C Hamano argued against flipping the default, saying the original commit made the comparison opt-in pending evidence that people like it, and that the change would surprise existing users. Harald Nordgren countered that most users have their push branch as upstream via `git push -u` and so miss information about upstream. Junio C Hamano suggested that whatever encourages `-u` could also suggest setting the configuration.

### `git status` holding `index.lock` only while writing
Mihai Cosma's two-patch series makes `git status` take `index.lock` only to write, and fixes a bug where under `index.skipHash` the check that stops status from overwriting a concurrently updated index passes for any index, silently reverting a `git add`. Junio C Hamano suggested that Jun Wu and Mihai Cosma send a single consolidated series; Jun Wu replied that his "Wait for the index lock instead of failing" series and this one are relatively independent -- his changes non-status commands while this mainly changes `status` -- and he prefers keeping them separate.

### Object storage extension in the object database
Patrick Steinhardt posted v3 of the seven-patch series introducing the object storage extension, configured via `extensions.objectStorage` with the same syntax as the ref storage extension. V3 splits the storage URI into an explicit `enum object_storage_format` plus a separate payload, adds `--object-storage-format=` to `git-init` and `git-clone` (local clones fall back to a non-local clone with a warning when formats differ), and propagates the format through `git-submodule` without passing payloads down.

## In brief
- Patrick Steinhardt reviewed Igor Wiedler's v3 fix for the connectivity check with alternates in deepening fetches and said it looks good with no further reroll needed.
- Karthik Nayak discussed Patrick Steinhardt's review of his v4 patch advertising fetched refs as haves when backfilling tags, explaining the `negotiation_include_tips` freeing logic and an existing leak in remote-helper transports he will address in a precursor commit.
- Tamir Duberstein posted v3 of the series letting `git ls-files` reuse and update the untracked cache, releasing the index lock before printing paths and skipping the optional index write with `--debug`.
- Hannes Brandstätter-Müller reported that `git apply` fails with "error: renamed: wrong type" on a pure rename followed by an edit under `core.fileMode=false`, diagnosed a zero `previous->new_mode` in `check_preimage()`, and attached a tentative fix with a regression test.
- Thomas Bachem posted v2 of two hook configuration changes -- `hook.<event>.enabled=false` now also suppresses the hooks-directory hook, and `.enabled`/`.parallel` accept a missing value as true -- and Junio C Hamano said he will queue the series.
