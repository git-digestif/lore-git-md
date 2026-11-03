# Git mailing list daily digest for 2026/11/02

## The day in brief

The Git mailing list saw significant activity around stalled patch series, with maintainers issuing warnings about imminent removals from integration branches due to lack of review. Key developments included substantive reviews of the `parse-options` early scanning API, the `git range-diff --matched-only` feature, and the reference-transaction hook regression fix. Process discussions about review bottlenecks and AI-assisted contributions gained traction, with maintainers emphasizing the need for human judgment in conceptual reviews.

## Notable threads

### [PATCH 0/7] line-log: support range-scoped diff stat, whitespace check, and -G pickaxe
**What changed:** Junio C Hamano warned that the v3 series will be dropped from `seen` unless reviewers step up within the next few weeks.

**Problem/goal:** Extend `git log -L` to support range-scoped diff stat formats, whitespace checking, and the `-G` pickaxe.

**Subsystem:** Line-log (`-L` option)

**Impact:** The series is at risk of abandonment despite being technically complete and addressing all prior feedback. The lack of reviewer engagement threatens to derail this enhancement to Git's line-range filtering capabilities.

---

### [PATCH v5] sequencer: handle fixup/squash commits that become empty
**What changed:** Phillip Wood provided a substantive review of v5, identifying additional improvements for clarity, error handling, edge cases, and correctness.

**Problem/goal:** Fix a long-standing edge case where `fixup!`/`squash!` commits that cancel out their target commit are not handled according to the `--empty` option.

**Subsystem:** Sequencer (`--autosquash` logic)

**Impact:** The review is actionable and emphasizes that changes to `commit_staged_changes()` are necessary for correctness. The patch remains ready for merging once the requested changes are addressed.

---

### [PATCH v7] git repo info: add path-related keys
**What changed:** Junio C Hamano issued a final warning that the v7 series will be dropped from `seen` unless reviewers engage. Justin Tobler identified a test correctness issue in patch 2/8 and provided surface-level reviews of patches 3/8, 4/8, 6/8, and 8/8.

**Problem/goal:** Expose filesystem locations of repository components via `git repo info`.

**Subsystem:** Repository information command

**Impact:** The series is at critical risk of abandonment despite being mechanically complete and correctness-fixed. The test correctness issue in patch 2/8 must be addressed before the series can progress.

---

### What's cooking in git.git (Jul 2026, #12)
**What changed:** Junio C Hamano proposed a mechanical policy to eject unreviewed topics from `seen` after four weeks. Patrick Steinhardt endorsed the policy, and Phillip Wood proposed explicitly stating that ejected topics can be re-queued if interest later resurfaces.

**Problem/goal:** Address the backlog of unreviewed topics in the `seen` integration branch.

**Subsystem:** Integration pipeline

**Impact:** The new policy aims to balance reviewer productivity with quality control. The discussion highlights systemic challenges in Git's review process, including the tension between contributor volume and reviewer bandwidth.

---

### [PATCH v2] hex-ll: restrict hex object IDs to lowercase only
**What changed:** Junio C Hamano requested final review to confirm the v2 RFC series aligns with prior discussion.

**Problem/goal:** Enforce lowercase-only parsing of hex object IDs in Git 3.0 to address security and compatibility issues.

**Subsystem:** Object ID parsing

**Impact:** The series is ready for integration into `next` if no new issues are raised. The breaking change targets Git 3.0 and remains gated behind `WITH_BREAKING_CHANGES`.

---

### [PATCH v2] contrib/credential/libsecret: avoid crash and silent data loss
**What changed:** Junio C Hamano warned that the topic may be dropped from `seen` without further feedback.

**Problem/goal:** Fix a crash and silent data loss in the `contrib/credential/libsecret` credential helper.

**Subsystem:** Credential helpers

**Impact:** The patch addresses a real issue but remains stalled due to lack of review participation. The procedural blocker threatens to delay the fix despite its clear value.

---

### [PATCH v6] parse-options: new sub-API for early argument scanning
**What changed:** Christian Couder posted v4 of the series, addressing all prior feedback including validation checks, ephemeral `option` pointer documentation, and test coverage for the `git fast-import` bugfix.

**Problem/goal:** Introduce a minimal, reusable `parse-options` sub-API (`early_scan_options()`) to replace fragile hand-rolled early argument scans.

**Subsystem:** Argument parsing

**Impact:** The series is ready for `next`, with the `git fast-import` bugfix and preparatory refactorings now uncontroversial. The core API includes validation checks, clearer documentation, and thorough test coverage.

---

### [PATCH v3 0/2] doc: sync command-list.txt with non-command manual pages
**What changed:** Julia Evans identified a design inconsistency in `lint-manpages.sh`'s variable extraction, suggesting consolidation to `Documentation/Makefile` for clarity.

**Problem/goal:** Ensure `command-list.txt` stays in sync with non-command manual pages.

**Subsystem:** Documentation

**Impact:** The review targets a potential source of future bugs and proposes a cleaner architectural approach. The patch author has not yet responded, so the discussion remains open.

---

### [PATCH v3] git-p4: prevent shell injection from user-supplied commit ID
**What changed:** Junio C Hamano warned that the patch will be dropped from `seen` unless reviewers step up.

**Problem/goal:** Fix a shell injection vulnerability in `git-p4.py`'s `applyCommit()` function.

**Subsystem:** `git-p4` (Git-Perforce bridge)

**Impact:** The patch is well-motivated and addresses a real security issue but remains stalled due to lack of review participation.

---

### [PATCH] rev-list: ignore replacements during connectivity check
**What changed:** Kristofer Karlsson posted the patch, and Junio C Hamano endorsed the core idea but suggested an alternative approach: checking connectivity for both original and replacement objects.

**Problem/goal:** Align `git rev-list`'s connectivity check with other commands by ignoring replacement refs.

**Subsystem:** Connectivity checks

**Impact:** The patch addresses an inconsistency that can cause packs to pass connectivity checks but later fail `fsck`. The discussion centers on design intent rather than implementation.

---

## In brief

- **[PATCH 0/2] fetch: configurable submodule fetch errors:** Junio C Hamano issued a second call for reviewers, warning that the topic will be discarded from `seen` unless reviewers engage.
- **[PATCH v2] submodule: fix crash and error handling in branch --recurse-submodules:** Junio C Hamano warned that the series will be dropped from integration branches unless reviewers step up.
- **[PATCH v2] git-svn: quiet migration noise and make build optional:** Junio C Hamano warned that the series will be dropped from `seen` unless reviewers step up.
- **[PATCH] gitremote-helpers: document `option pushcert` "if-asked" value:** Junio C Hamano reviewed the patch and marked it ready for `next`.
- **[PATCH 0/6] parse-options: new sub-API for early argument scanning:** Christian Couder posted v4 of the series, addressing all prior feedback.
- **[PATCH v3 0/2] doc: sync command-list.txt with non-command manual pages:** Julia Evans identified a design inconsistency in `lint-manpages.sh`.
- **[PATCH] add -i: make init/clear_add_i_state() static:** No noteworthy new developments.
- **[PATCH] read-cache: make refresh_cache_entry() static again:** No noteworthy new developments.
- **[Outreachy] GIT_DIR inherited by shell aliases in linked worktrees:** Priestess (Cynthia) confirmed the issue and identified `GIT_DIR` inheritance as the root cause.
- **[PATCH] git-svn: prevent shell injection from user-supplied commit ID:** Junio C Hamano warned that the patch will be dropped from `seen` unless reviewers step up.
- **[PATCH 0/2] submodule: fix crash and error handling in branch --recurse-submodules:** Junio C Hamano warned that the topic will be dropped unless reviews materialize.
- **[PATCH 0/8] ci: consolidate test environment variables:** Phillip Wood explicitly stated the series is "ready for `next`" following his approval.
- **[PATCH] odb.c: skip self-referential alternates during quarantine:** Kristofer Karlsson posted v2 of the patch, repurposing `GIT_QUARANTINE_ENVIRONMENT` to avoid fragile path comparisons.
- **[PATCH] add -i: make init/clear_add_i_state() static:** No noteworthy new developments.
- **[PATCH] read-cache: make refresh_cache_entry() static again:** Junio C Hamano confirmed the final version of the patch will adopt Patrick Steinhardt's suggested reordering of functions.
- **[Outreachy] GIT_DIR inherited by shell aliases in linked worktrees:** No noteworthy new developments.
- **[PATCH] diff-files: convert to parse-options API:** The author withdrew the patch after Junio C Hamano raised concerns about the value of the refactoring for a plumbing command.
- **[Outreachy] environment variable handling in Git worktrees:** No noteworthy new developments.
- **[PATCH] notes-utils: pass repository parameter explicitly:** Patrick Steinhardt warned that the patch still reads as AI-generated and threatened to ignore future patches unless the commit message is genuinely rewritten.
- **[PATCH 0/6] stash: improve performance by eliminating duplicate work:** Phillip Wood pointed to an overlapping series in `seen` and invited the author to review it.
- **[PATCH 0/2] maintenance: Trace2 event + lock-handling consistency:** Patrick Steinhardt requested mechanical refinements: splitting the patch into two, deduplicating error-message formatting, and simplifying the Trace2 event's `reason` field.
- **[PATCH] doc: sync minimum Perl version with code:** Junio C Hamano requested a wording correction to align the documentation with the code's use of `require v5.26`.
- **[PATCH] diff-auto_refresh_index: migrate to repo_config_values:** Ebele Lynda Okolo posted the first iteration of the patch, migrating the `diff_auto_refresh_index` global variable into the `repo_config_values` struct.
- **[PATCH 0/6] odb: move loose object backend into its own source file:** Patrick Steinhardt posted v1 of the series, moving loose-object backend logic from `object-file.c` into `odb/source-loose.c`.
- **[Outreachy] libifying pathspec handling in find_pathspecs_matching_skip_worktree():** Srujith Poojary proposed two approaches to eliminate a `the_repository` global access in the pathspec subsystem.
- **[PATCH] Documentation/Governance.adoc:** Antonin Delpeuch posted the initial draft, and Junio C Hamano requested line-wrapping improvements and provided clarifications on governance details.
- **[PATCH] rev-list: ignore replacements during connectivity check:** Kristofer Karlsson posted the patch, and Junio C Hamano endorsed the core idea but suggested an alternative approach.
- **[Git Rev News #140]:** Christian Couder announced the publication of *Git Rev News* edition #140.
- **[.cirrus.yml]:** Tuomas Ahola asked whether the `.cirrus.yml` file is still in use, given that Cirrus CI appears defunct.