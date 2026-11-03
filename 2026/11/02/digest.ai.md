# Git mailing list daily digest for 2026/11/02

## The day in brief
The Git mailing list saw **137 emails** today, with key developments including: a **maintainer warning** that several stalled patch series will be dropped from `seen` unless reviewers step up; **substantive reviews** on the `parse-options` early scanning API, `git stash` performance improvements, and `git range-diff --matched-only`; **process discussions** about AI-assisted contributions and review bottlenecks; and **new patches** for repository-scoped configuration, ODB abstraction, and governance documentation.

## Notable threads

### [PATCH 0/7] line-log: support range-scoped diff stat, whitespace check, and -G pickaxe
**What changed?**
Junio C Hamano warned that this series—extending `git log -L` to support `--stat`, `--check`, and `-G`—will be **dropped from `seen`** unless reviewers engage within the next few weeks. The series is technically complete (v3) and addresses all prior feedback, but lacks substantive review.

**Why it matters**
The feature would make `git log -L` more consistent with standard diff options, improving usability for line-range filtering. The series is at risk of abandonment despite its readiness, highlighting a recurring challenge in attracting reviewer attention for niche features.

**Key details**
- **Files touched**: `diff.c`, `diffcore-pickaxe.c`, `revision.c`, `xdiff-interface.c`, `Documentation/line-range-options.txt`, `t/t4211-line-log.sh`
- **New behavior**: `--stat`, `--check`, and `-G` now honor tracked line ranges, with exclusions for `--dirstat` and `-S`
- **Test coverage**: Extensive updates to `t/t4211-line-log.sh` for stat counts, whitespace errors, `-G` matches, and edge cases

---

### [PATCH v5] rebase: fix autosquash behavior with `--empty=drop`
**What changed?**
Phillip Wood provided a **substantive review** of v5, identifying additional improvements for clarity, error handling, edge cases, and correctness in `commit_staged_changes()`. The review emphasizes that changes to this function are **necessary for correctness**.

**Why it matters**
The patch fixes a long-standing edge case where `fixup!`/`squash!` commits that cancel out their target are not handled according to `--empty` options. The fix enables workflows like automatically dropping both a commit and its revert when `--empty=drop` is used.

**Key details**
- **Files touched**: `sequencer.c`, `Documentation/git-rebase.adoc`, `t/t3415-rebase-autosquash.sh`, `t/t5407-post-rewrite-hook.sh`
- **New symbols**: `is_amended_head_empty()`, `enum fixup_target`, `PICK_RESULT_DROPPED_HEAD`, `rebase_path_fixup_target()`
- **Behavior change**: Only commits that *become* empty (not those picked empty) are subject to `--empty` handling
- **Test coverage**: Comprehensive tests for all `--empty` modes (`drop`, `keep`, `stop`) and edge cases

---

### [PATCH v7 0/8] git repo info: add path-related keys
**What changed?**
Junio C Hamano issued a **final warning** that the series will be dropped from `seen` unless reviewers step up. Justin Tobler provided **substantive reviews** of multiple patches, identifying:
1. A **test correctness issue** in patch 2/8 (bisectability break)
2. **Design questions** about `/dev/null` handling in `path.hooks` and grafts inclusion
3. **Surface-level feedback** on `path.cdup` implementation and test hygiene

**Why it matters**
The series exposes filesystem locations of repository components (e.g., hooks directory, index file) in a scriptable format via `git repo info`. It’s feature-complete and well-tested but stalled due to lack of reviewer bandwidth.

**Key details**
- **Files touched**: `builtin/repo.c`, `Documentation/git-repo.adoc`, `t/t1900-repo-info.sh`, `submodule.c`/`submodule.h`
- **New keys**: 24 path-related keys with `.absolute`/`.relative` suffixes, plus `path.git-prefix` and `path.cdup`
- **Open questions**:
  - Should `path.hooks.relative` return `/dev/null` verbatim or a relative path when `core.hooksPath` is set to `/dev/null`?
  - Should `path.grafts` keys exist given grafts are deprecated?

---

### Integration pipeline policy: `seen` branch management
**What changed?**
Junio C Hamano proposed a **mechanical policy** to eject unreviewed topics from `seen` after 4 weeks. The discussion explored two approaches:
1. **Ejection after 4 weeks** (current policy, aligned with "Waiting for response" timeout)
2. **Strict intake** (only queue topics with prior substantive review)

Phillip Wood proposed **explicitly stating that ejection is reversible** to reduce contributor anxiety.

**Why it matters**
The policy aims to balance reviewer productivity with quality control. The discussion revealed tensions between:
- **Contributor experience** (avoiding unceremonious ejections)
- **Maintainer workload** (reducing backlog)
- **Reviewer productivity** (low-friction access to patches)

**Key details**
- **Current policy**: Topics expire after 4 weeks without positive feedback
- **Proposed refinements**:
  - Document reversibility of ejection
  - Clarify "Will merge to 'next'?" state
  - Consider hybrid eligibility models for voting

---

### [PATCH v4 0/4] parse-options: new sub-API for early argument scanning
**What changed?**
Christian Couder posted v4 of the series, addressing all prior feedback:
- Added **validation checks** inside `early_scan_options()`
- Clarified **ephemeral `option` pointer** documentation
- Replaced `if`-statement chain with `switch` in `parse_long_opt()`
- Improved **test coverage** and documentation

**Why it matters**
The API replaces fragile hand-rolled early argument scans in Git commands, fixing a real bug in `git fast-import` where `--allow-unsafe-features` was silently ignored when preceded by `--depth 5`.

**Key details**
- **Files touched**: `parse-options.h`, `parse-options.c`, `fast-import.c`, test helpers, `t/t0040-parse-options.sh`
- **New functions**: `early_scan_options()`, `find_early_scan_option()`
- **Behavior change**: Fixes `git fast-import --depth 5 --allow-unsafe-features`

---

## In brief
- **[PATCH] git-p4: prevent shell injection**: Junio warned the v2 patch will be dropped from `seen` without review. The fix addresses a security vulnerability in the `--commit` option.
- **[PATCH 0/2] submodule: fix crash in branch --recurse-submodules**: Junio warned the series will be dropped from `seen` unless reviewers step up. The fix addresses a BUG in the experimental `submodule.propagateBranches` feature.
- **[PATCH v2] git-svn: quiet migration noise**: Junio warned the series will be dropped from `seen` due to lack of review. The patch suppresses spurious migration messages and adds a `NO_GIT_SVN` Makefile knob.
- **[PATCH] doc: sync command-list.txt with non-command manual pages**: Julia Evans identified a **design inconsistency** in `lint-manpages.sh` (mixing `../Makefile` and `Documentation/Makefile` for variable extraction).
- **[PATCH] rev-parse: add advice for shallow history**: Junio warned the patch will be dropped from `seen` unless reviewers step up. The patch adds `advice.shallowHistory` for `<rev>~N`/`<rev>^N` failures in shallow clones.
- **[PATCH] odb.c: skip self-referential alternates during quarantine**: Kristofer Karlsson posted v2, repurposing `GIT_QUARANTINE_ENVIRONMENT` to avoid fragile path comparisons. The patch fixes a performance regression in `git rev-list --alternate-refs`.
- **[PATCH 0/6] odb: move loose object backend into its own source file**: Patrick Steinhardt posted v1, isolating loose-object backend logic into `odb/source-loose.c` as part of the ODB abstraction effort.
- **[PATCH] add -i: make init/clear_add_i_state() static**: Patrick Steinhardt confirmed the lack of `notes.rewriteref` support in `git history fixup` is a known limitation.
- **[PATCH] read-cache: make refresh_cache_entry() static again**: Junio adopted Patrick’s suggestion to reorder functions to avoid a forward declaration.
- **[PATCH] rev-list: ignore replacements during connectivity check**: Junio suggested an alternative approach: checking connectivity for both original *and* replacement objects.
- **[PATCH] notes-utils: pass repository parameter explicitly**: Patrick Steinhardt warned the patch still reads as AI-generated and threatened to ignore future patches unless the commit message is genuinely rewritten.
- **[PATCH 0/2] maintenance: Trace2 event + lock-handling consistency**: Patrick Steinhardt requested mechanical refinements: splitting the patch, deduplicating error-message formatting, and simplifying the Trace2 event format.
- **[PATCH] diff-files: convert to parse-options API**: The author withdrew the patch after Junio questioned the value of the refactoring for a plumbing command.
- **[PATCH] Documentation/Governance.adoc**: Junio requested line-wrapping improvements and clarifications on project scope, PLC transitions, and security list access.

## Process discussions
Today’s emails included **extensive discussion** about:
1. **AI-assisted contributions**: Junio warned against incentivizing low-effort or AI-generated reviews via quota systems, emphasizing that human review of conceptual direction is non-negotiable.
2. **Review bottlenecks**: Proposals included:
   - Pre-review automation (e.g., Sashiko, Codecov)
   - "Review quota" policies (contributors must review others’ work)
   - Deferring nitpicking until core ideas are settled
3. **Voting eligibility**: Jeff King proposed a hybrid model combining an algorithmic threshold with a human-curated list for edge cases. Patrick Steinhardt raised governance questions about the list’s legitimacy and transparency.