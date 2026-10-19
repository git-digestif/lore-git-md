# Git mailing list daily digest for 2026/10/18

## The day in brief
The Git mailing list saw final revisions of two long-running patch series: Tyler Cipriani’s `--force-if-includes` bugfix (v7) and Harald Nordgren’s reflog usability improvements (v2). Torsten Bögershausen proposed a structural change to the CRLF conversion fix, while Harald Nordgren’s experiment with `git range-diff` for rebase detection revealed a 20× performance penalty. Outreachy microprojects and small bugfixes rounded out the day’s activity.

## Notable threads

### CRLF conversion size-trusting bug: structural proposal to avoid racy Git interference
**What changed?**
Torsten Bögershausen proposed moving the `would_convert_to_git()` call from `ie_modified()` to `ie_match_stat()` in `read-cache.c` to avoid interfering with Git’s racy Git logic (where `CE_MATCH_RACY_IS_DIRTY` is set). He reported successful testing with a real repository where a clean filter replaces tabs with spaces, noting that `git status` no longer incorrectly reports modifications.

**Why it matters**
The CRLF conversion bug affects macOS and other environments where tools rewrite files with CRLF line endings, causing commands like `git status` to falsely report modifications. The current fix (v5) adds a conversion check in `ie_modified()`, but Junio C Hamano raised concerns about its interaction with racy Git logic. Torsten’s proposal relocates the check to `ie_match_stat()`, which could resolve the racy Git interference but may reintroduce performance overhead in a broader set of commands. The architectural debate remains unresolved: Johannes Sixt argues the fix should target `git add` (correcting the index’s recorded size), while Curtis Allen Smith maintains it belongs in index refresh operations.

**Key details**
- Subsystem: `read-cache.c` (index refresh logic)
- Change category: bugfix (platform compatibility)
- Impact: affects all commands that refresh the index (e.g., `git status`, `git rebase`, `git pull --rebase`)
- Open question: whether the relocation to `ie_match_stat()` resolves the racy Git concerns or reintroduces performance trade-offs

---

### `--force-if-includes` bugfix: v7 final revision posted
**What changed?**
Tyler Cipriani posted v7 of the `--force-if-includes` bugfix series, signaling this is the final version unless new technical issues are raised. The series fixes two critical flaws: (1) `--force-if-includes` incorrectly checks the reflog of the *local* branch matching the *remote* destination name (e.g., `refs/heads/main`) instead of the branch actually being pushed (e.g., `refs/heads/src` in `src:main`), and (2) it unnecessarily rejects fast-forward pushes of non-branch refs (tags, detached HEADs). The v7 series includes:
1. **Patch 1/3**: Fixes the core logic in `remote.c` to consult the reflog of the pushed ref (`ref->peer_ref->name`), with 81 lines of new test coverage.
2. **Patch 2/3**: Introduces a new `ref->unverifiable` flag and actionable advice (`advice.pushRefUnverifiable`) for pushes lacking a reflog (e.g., detached HEAD, tags).
3. **Patch 3/3**: Exempts fast-forward pushes from `--force-if-includes` checks, resolving the existing bug where fast-forwards of tags or detached HEADs were rejected.

**Why it matters**
The series addresses both **data loss risks** (allowing pushes that clobber remote history) and **false rejections** (blocking valid pushes). The new advice mechanism resolves a long-standing UX flaw by providing unambiguous guidance (`--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`) for unverifiable refs. All prior review feedback has been addressed, including variable naming clarity (`needs_force_reject_reason`), edge-case handling (deletions, unresolved `HEAD`), and test coverage completeness.

**Key details**
- Files touched: 12 files, including `remote.c`, `advice.c`, and `t/t5533-push-cas.sh` (108 lines of new test coverage)
- New symbols: `advice.pushRefUnverifiable`, `ADVICE_PUSH_REF_UNVERIFIABLE`, `ref->unverifiable`, `REF_STATUS_REJECT_UNVERIFIABLE`, `REJECT_REF_UNVERIFIABLE`
- Behavior change: force-pushes of non-branch refs are now rejected by default (with actionable advice), while fast-forward pushes of non-branch refs are allowed
- Status: technically complete and ready for merge; trivial test conflict with `as/push-force-if-includes-no-reflog` remains

---

### Documentation cross-reference modernization: l10n support rationale
**What changed?**
Jean-Noël Avila clarified that the two-part AsciiDoc link syntax (`<<target,text>>`) is necessary for supporting translated man pages. The link text must be decoupled from the section title to accommodate grammatical or idiomatic requirements in other languages, strengthening the case for the ongoing cross-reference modernization effort.

**Why it matters**
The cross-reference modernization patch (v2) replaces informal references (e.g., "see EXAMPLES below") with formal AsciiDoc cross-references to improve HTML navigation. The patch is approved and queued in `next` but not yet merged to `master`. Jean-Noël’s contribution adds a new technical dimension: the two-part form is not just a workaround for rendering quirks but a requirement for l10n support. This justification may influence the project’s stance on the anchor style trade-off (manual vs. auto-generated anchors), though the patch itself remains unchanged.

**Key details**
- Files: 38 files in `Documentation/`, including `git-rebase.adoc`, `git-grep.adoc`, and `git-push.adoc`
- Change category: documentation (navigation improvement)
- Impact: no functional change to man page content; improves HTML navigation and l10n support
- Open question: whether to prioritize link stability (manual anchors) or readability (auto-generated anchors)

---

### `git branch --delete-merged`: rebase detection performance findings
**What changed?**
Harald Nordgren reported on a local experiment using `git range-diff` for rebase detection in the `--delete-merged` feature. The range-diff approach correctly identifies the same 14 branches to delete as the v2 implementation but is **20× slower** (10 seconds vs. 0.5 seconds for 65 branches). He also clarified that `git range-diff` pairs commits based on diff content alone, with log messages and authorship appearing only in the output and not influencing the matching process.

**Why it matters**
The `--delete-merged` series aims to extend `git branch --delete-merged` to recognize branches whose changes (but not their tips) have been integrated via squash merges, rebases, or patch-based workflows. Junio C Hamano proposed reusing `git range-diff`’s commit-matching logic for rebase detection, but Harald’s experiment reveals a critical performance trade-off. The current tree-based diffing approach is fast but may miss edge cases (e.g., edited commit messages), while `git range-diff` is more thorough but impractical for the `--delete-merged` use case. The discussion now centers on whether to accept the limitations of the faster approach or explore alternatives like patch IDs (proposed by D. Ben Knoble).

**Key details**
- Subsystem: `builtin/branch.c` (branch deletion logic)
- Change category: feature (repository maintenance)
- Performance impact: `git range-diff` is 20× slower than the current tree-based diffing approach
- Open question: whether to adopt patch IDs or another middle ground for rebase detection

---

### `git reflog` usability improvements: v2 posted
**What changed?**
Harald Nordgren posted v2 of the reflog `--operations` and `--historical-branch` series, addressing feedback on commit message clarity and `--historical-branch` output formatting. The series now includes 15 files changed and a 198-line test script (`t/t1414-reflog-walk.sh`). Key changes:
- **Patch 1/3**: Preparatory refactoring to replace hard-coded reflog message strings with `#define` constants in `reflog.h`.
- **Patch 2/3**: Implements `--operations` to collapse multi-step operations (e.g., rebase, stash) into single entries.
- **Patch 3/3**: Implements `--historical-branch` to decorate entries with the branch name at recording time (branch names now appear in parentheses after the operation description, e.g., `HEAD@{0}: (topic) commit: add tests`).

**Why it matters**
The series addresses long-standing usability pain points in the reflog subsystem. `--operations` reduces noise from intermediate reflog entries, while `--historical-branch` provides branch context for each entry, eliminating the need to manually trace "checkout: moving from" entries. The v2 patches address prior feedback on output formatting and commit message clarity, though the broader architectural question—whether Git’s core should replicate functionality already available in tools like `tig`—remains unresolved.

**Key details**
- Files touched: 15 files, including `reflog-walk.c`, `revision.c`, and `t/t1414-reflog-walk.sh` (198 lines of new test coverage)
- New symbols: `#define` constants in `reflog.h` (`REFLOG_SWITCH_PREFIX`, `REFLOG_RENAME_PREFIX`, etc.), `user_operation` flag, `historical_branch` flag
- Change category: feature (usability improvement)
- Impact: additive options requiring `--walk-reflogs`; no changes to default behavior

---

### `branch.autoSetupMerge=inherit`: user workflows and alternative designs
**What changed?**
Harald Nordgren proposed prioritizing inheritance from the *current* branch’s tracking configuration (rather than the starting point’s) when creating a new branch. If the current branch already tracks a remote (e.g., `origin/master`), the new branch would inherit *that* tracking relationship by default, with a fallback to tracking the starting point directly only if the current branch has no tracking. Phillip Wood described workflows where the current behavior (doing nothing for non-tracking starting points) may be preferable, such as creating a local branch from a pushed-but-not-tracked branch and preferring to track a different upstream (e.g., `origin/main`).

**Why it matters**
The `branch.autoSetupMerge=inherit` setting fails to set up tracking when the starting point is a remote-tracking branch (e.g., `origin/main`), unlike the default (`true`) behavior. The discussion has expanded to explore alternative designs, including Junio C Hamano’s proposed modes (`inherit-or-fail`, `inherit-or-direct`) for non-tracking starting points. Harald’s new proposal attempts to reconcile diverse user expectations by making the current branch’s tracking configuration the primary source of inheritance, but it risks complicating the mental model for users who expect tracking to flow from the starting point.

**Key details**
- Subsystem: `branch.c` (branch creation and tracking logic)
- Change category: bugfix/feature (tracking configuration)
- Open question: whether to adopt Harald’s current-branch inheritance proposal, Junio’s modes, or retain the status quo
- Impact: affects branch creation workflows (e.g., `git switch -c new-branch origin/main`)

---

## In brief
- **[2026/10/18/01-19-59 by Junio C Hamano]** Ejected Igor Wiedler’s second patch from `seen` due to a Windows CI test failure caused by POSIX/Windows path incompatibility in the test. The failure is mechanical (path normalization) and not a design flaw.
- **[2026/10/18/02-42-05 by ceo]** Proposed clarifying the `<name>` argument in `git remote add --help` by adding a two-line explanation that "origin" is the default name assigned by `git clone` but that any valid name can be used.
- **[2026/10/18/04-55-01 by Srujith A Poojary]** Posted a test modernization patch replacing a pipe in `t/t0001-init.sh` with a two-step process to avoid masking the exit status of `git version`.
- **[2026/10/18/13-25-41 by Omri Sarig]** Posted a patch removing outdated references to incompatible options (`--allow-empty-message`, `--ignore-date`, `--signoff`) in the `git-rebase` man page, following up on prior commits that removed these options from the code.
- **[2026/10/18/14-21-29 by Olof Fairbrook]** Posted a bugfix patch for `git bundle create` to avoid freeing uninitialized stack memory when given an unrecognized argument.
- **[2026/10/18/15-24-26 by Olof Fairbrook]** Posted a bugfix patch for the config subsystem to correctly handle high-bit bytes (0x80–0xff) in in-memory config blobs, fixing a long-standing issue where 0xff was misinterpreted as EOF and UTF-8 BOMs triggered spurious errors.
- **[2026/10/18/16-31-12 by Shashank S]** Posted a patch removing the implicit dependency on `the_repository` in `prune_packed_objects()` by converting it to accept an explicit `struct repository *` parameter.
- **[2026/10/18/17-59-44 by Aniket Gupta]** Asked whether the fsck pluralization microproject is still available and offered to send the patch if unclaimed.
- **[2026/10/18/20-53-05 by Andy Koppe]** Posted a patch fixing a Makefile bug where the Git build fails if the user has a custom Cargo target directory, by explicitly setting `--target-dir target` for Cargo commands.
- **[2026/10/18/23-55-07 by Colin Hinton]** Posted a patch deferring validation of `remote.<name>.followRemoteHEAD` until it is actually needed (during `git fetch`), eliminating spurious warnings from commands that load remote configuration but never use this setting. The patch also unifies the parsing logic for both `remote.<name>.followRemoteHEAD` and `fetch.followRemoteHEAD`.