# Git mailing list daily digest for 2026/10/18

## The day in brief
Tyler Cipriani posted the final v7 of a critical `--force-if-includes` bugfix series, signaling readiness for merge. Harald Nordgren’s rebase-detection experiment revealed `git range-diff` is 20× slower than tree-based diffing, complicating the `--delete-merged` feature design. Torsten Bögershausen proposed relocating a CRLF conversion check to avoid racy Git interference, reopening the architectural debate for macOS compatibility fixes.

## Notable threads

### `--force-if-includes` bugfix series finalized
[2026/09/04/21-01-20] Tyler Cipriani posted v7 of the three-patch series fixing the `--force-if-includes` safety mechanism in `git push`. The series is now technically complete, with all prior feedback resolved and only minor typographical fixes and rebase adjustments since v6. The author explicitly signals this is the final version unless new technical issues are raised.

The core problem remains: `--force-if-includes` incorrectly checks the reflog of the *local* branch whose name matches the *remote* destination (e.g., `refs/heads/main`), rather than the branch actually being pushed (e.g., `refs/heads/src` in `src:main`). This causes both false rejections (blocking valid pushes when the local branch is behind) and data loss risks (allowing pushes that clobber remote history). Additionally, the feature unnecessarily rejects fast-forward pushes of non-branch refs (tags, detached HEADs), breaking workflows.

The v7 series consists of three patches:
1. **Patch 1/3**: Fixes the core logic in `remote.c` to consult the reflog of the pushed ref (`ref->peer_ref->name`). Includes 81 lines of new test coverage in `t/t5533-push-cas.sh` for edge cases like differently named branches, HEAD-based pushes, orphan branches, detached HEADs, and tags.
2. **Patch 2/3**: Introduces a new `ref->unverifiable` flag and actionable advice (`advice.pushRefUnverifiable`) for pushes lacking a reflog (e.g., detached HEAD, tags). Updates all transport layers to emit consistent advice, instructing users to use `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`.
3. **Patch 3/3**: Exempts fast-forward pushes from `--force-if-includes` checks, resolving the existing bug where fast-forwards of non-branch refs were incorrectly rejected.

The series touches 12 files and introduces new symbols like `advice.pushRefUnverifiable`, `ADVICE_PUSH_REF_UNVERIFIABLE`, `ref->unverifiable`, `REF_STATUS_REJECT_UNVERIFIABLE`, and `REJECT_REF_UNVERIFIABLE`. The behavior change is significant but well-justified: force-pushes of non-branch refs are now rejected by default (with actionable advice), while fast-forward pushes of non-branch refs are allowed, fixing a long-standing bug.

All prior review feedback has been addressed, including variable naming clarity, commit message typos, edge-case handling, and advice actionability. The series is ready for merge and has no unresolved technical concerns. The author offers to send a rebased v8 if needed to resolve a remaining trivial test conflict with `as/push-force-if-includes-no-reflog`, but this is not a blocker.

### CRLF conversion check relocation proposed
[2026/01/11/19-51-49] Torsten Bögershausen proposed moving the `would_convert_to_git()` call from `ie_modified()` to `ie_match_stat()` in the ongoing thread about Curtis Allen Smith’s v5 patch for the CRLF conversion size-trusting bug in `read-cache.c`. The proposal aims to avoid interfering with Git’s racy Git logic, where `CE_MATCH_RACY_IS_DIRTY` is set.

Torsten also reported successful testing with a real repository where a clean filter replaces tabs with spaces, noting that `git status` no longer incorrectly reports modifications. The suggestion to relocate the conversion check is technically straightforward but touches on the same architectural tension that has run through the thread: whether the fix should target index refresh operations (`ie_modified()`) or the lower-level stat comparison (`ie_match_stat()`).

The proposal does not address the broader architectural objection from Johannes Sixt (that the index’s recorded size is a cache of stat information and should be corrected during `git add` rather than in refresh operations) or Junio C Hamano’s concerns about performance trade-offs. The relocation could reintroduce performance overhead in a broader set of commands, as `ie_match_stat()` is used by more operations than `ie_modified()`. The thread remains unresolved, with no clear path forward yet.

### Rebase detection performance findings
[2026/09/29/07-30-30] Harald Nordgren reported on a local experiment using `git range-diff` for rebase detection in the `--delete-merged` feature. The experiment revealed that `git range-diff` correctly identifies the same 14 branches to delete as the v2 implementation but is 20× slower (10 seconds vs. 0.5 seconds for 65 branches). Harald also clarified that `git range-diff` pairs commits based on diff content alone, with log messages and authorship appearing only in the output and not influencing the matching process.

This contradicts Junio C Hamano’s assumption that `git range-diff`’s commit-matching logic could be reused to improve rebase detection by incorporating log-message similarity and authorship. The performance penalty makes `git range-diff` impractical for the `--delete-merged` use case, where speed is critical for usability. The discussion now hinges on whether a middle ground exists—such as using patch IDs (as proposed by D. Ben Knoble) or selectively applying `git range-diff`’s logic as a fallback—or whether the project should accept the limitations of the faster tree-based diffing approach.

Harald’s findings do not address Junio’s broader CLI redesign proposal (decoupling `--delete-merged` and `--delete-irrelevant`) or Phillip Wood’s call for clearer documentation of the algorithm’s limitations. These remain open questions, but the performance data will likely shape the next iteration of the series.

### `branch.autoSetupMerge=inherit` workflow feedback
[2026/10/16/07-02-45] Phillip Wood described workflows where the current behavior of `branch.autoSetupMerge=inherit` (doing nothing for non-tracking starting points) may be preferable. He provided two common scenarios: (1) when creating a local branch from a remote-tracking branch (e.g., `origin/main`), he *does* want the new branch to track the starting point, but (2) when creating a local branch from a pushed-but-not-tracked branch (e.g., `my-feature` pushed to `origin/my-feature`), he *does not* want the new branch to track the starting point—instead, he prefers to track something like `origin/main`.

This real-world example undermines the patch’s proposed behavior (tracking the starting point directly when no tracking exists) and suggests that the current behavior may actually be more intuitive for some users. Phillip’s feedback shifts the discussion from a technical bugfix to a usability question: what do users *expect* `inherit` to do when the starting point lacks tracking? His closing remark about the confusion around `git switch -c` tracking options hints that the entire tracking setup mechanism may need broader rethinking.

Harald Nordgren later proposed prioritizing inheritance from the *current* branch’s tracking configuration (rather than the starting point’s) when creating a new branch. This would address Phillip’s described workflow where users often want new branches to track the same upstream as their parent branch. Harald suggested retaining the original patch’s behavior (tracking the starting point directly) as a fallback only if the current branch has no tracking. This hybrid approach attempts to reconcile the two workflows but remains conceptual and lacks implementation details.

## In brief
- **[2026/10/18/12-32-15 by Jean-Noël AVILA]** Clarified that the two-part AsciiDoc link syntax (`<<target,text>>`) is necessary for supporting translated man pages, as it allows the link text to be decoupled from the section title to accommodate grammatical or idiomatic requirements in other languages.
- **[2026/10/18/01-19-59 by Junio C Hamano]** Ejected Igor Wiedler’s second patch from `seen` due to a Windows CI test failure caused by POSIX/Windows path incompatibility in the test. The failure is mechanical, and Jeff King has proposed two backward-compatible fixes.
- **[2026/10/18/18-27-14 by Harald Nordgren]** Posted v2 of the reflog `--operations` and `--historical-branch` series, addressing feedback on commit message clarity and `--historical-branch` output formatting (branch names now appear in parentheses after the operation description).
- **[2026/10/18/02-42-05 by ceo]** Proposed clarifying the `<name>` argument in `git remote add --help` by adding a two-line explanation that "origin" is the default name assigned by `git clone` but that any valid name can be used.
- **[2026/10/18/04-55-01 by Srujith A Poojary]** Posted a test modernization patch replacing a pipe in `t/t0001-init.sh` with a two-step process to avoid masking the exit status of `git version`.
- **[2026/10/18/13-25-41 by Omri Sarig]** Posted a patch removing outdated references to incompatible options (`--allow-empty-message`, `--ignore-date`, `--signoff`) in the `git-rebase` man page, following up on prior commits that removed these options from the code.
- **[2026/10/18/14-21-29 by Olof Fairbrook]** Posted a bugfix patch for `git bundle create` to avoid freeing uninitialized stack memory when given an unrecognized argument.
- **[2026/10/18/15-24-26 by Olof Fairbrook]** Posted a bugfix patch for the config subsystem to correctly handle high-bit bytes (0x80–0xff) in in-memory config blobs, fixing a long-standing issue where 0xff was misinterpreted as EOF and UTF-8 BOMs triggered spurious errors.
- **[2026/10/18/16-31-12 by Shashank S]** Posted a patch removing the implicit dependency on `the_repository` in `prune_packed_objects()` by converting it to accept an explicit `struct repository *` parameter.
- **[2026/10/18/20-53-05 by Andy Koppe]** Posted a patch fixing a Makefile bug where the Git build fails if the user has a custom Cargo target directory, by explicitly setting `--target-dir target` for Cargo commands.
- **[2026/10/18/23-55-07 by Colin Hinton]** Posted a patch deferring validation of `remote.<name>.followRemoteHEAD` until it is actually needed (during `git fetch`), eliminating spurious warnings from commands that load remote configuration but never use this setting.
- **[2026/10/18/17-59-44 by Aniket Gupta]** Asked whether the fsck pluralization microproject is still available and offered to send the patch if unclaimed.