# Git mailing list daily digest for 2026/09/08

## The day in brief
The `git history squash` feature reached v15, addressing all prior feedback and now cooking in `seen`. A critical design flaw was identified in the `receive-report` hook series, requiring a v9 reroll. Junio C Hamano critiqued modern commit messages for verbosity and urged compression. Several bugfixes and refactorings advanced, including fixes for `git bundle` with bitmaps, `git send-email` SMTP rate-limiting, and MIDX empty layer publication.

## Notable threads

### `git history squash` v15 now feature-complete
**What changed**: The `git history squash` feature, which folds a commit range into its oldest ancestor while preserving descendant history, reached v15. This version enforces strict case-sensitive matching for autosquash markers (rejecting uppercase OIDs) by replacing `istarts_with()` with `starts_with()` in `squash_check_autosquash_subject()`.

**Why it matters**: This standalone subcommand addresses a longstanding workflow gap by efficiently collapsing commit ranges without the repeated conflict stops of `git rebase -i`. The series is now technically complete, with all prior feedback addressed, including ref protection for local branches, the `--no-edit` workflow, and the default interactive message-editing workflow.

**Technical details**:
- Files touched: `builtin/history.c`, `sequencer.c`/`sequencer.h`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`, `t/t9902-completion.sh`
- New helpers: `first_parent_tree_oid()`, `resolve_squash_range()`, `squash_check_autosquash_subject()`, `setup_squash_revisions()`
- Key behaviors: shape-based validation, ref protection, autosquash handling, merge-parent preservation
- The series is marked "Will replace" by Junio since v7 and is now cooking in `seen`

---

### Critical design flaw identified in `receive-report` hook series
**What changed**: Junio C Hamano identified a critical design flaw in the second patch of the `receive-report` hook series. The `BUG()` call in the `switch` statement assumes the client’s requested protocol version is always valid, but clients may omit the `report-status` or `report-status-v2` capability entirely, leaving `version` as `REPORT_STATUS_UNKNOWN`.

**Why it matters**: This flaw could crash the server when handling malformed client input. The fix will replace the `default: BUG(...)` case with an explicit `REPORT_STATUS_UNKNOWN` case that `break`s, ensuring the code silently skips the status report when the client does not request one.

**Technical details**:
- Files touched: `builtin/receive-pack.c`
- New enum: `report_status_version` (values: `REPORT_STATUS_UNKNOWN`, `REPORT_STATUS_V0`, `REPORT_STATUS_V2`)
- The fix aligns with the existing behavior and avoids crashing the server over a client’s capability advertisement
- The series is queued in `next` but blocked from graduating to `master` until this fix is applied

---

### Junio critiques modern commit messages for verbosity
**What changed**: Junio C Hamano critiqued modern commit messages for growing longer with "irrelevant details" and urged compression, suggesting "Say the same thing in 1/3 of the words." He provided a concrete example of how the third patch in the auto-maintenance deferral series could be tightened.

**Why it matters**: This critique signals a shift in maintainer expectations for commit message style, prioritizing brevity and focus. Kristoffer Haugsbakk contextualized the trend, noting that recent commit messages increasingly resemble checklists and sometimes read like "code-to-English descriptions."

**Technical details**:
- The critique was framed as a general trend rather than a specific complaint about the auto-maintenance series
- Junio explicitly stated he would queue the series despite the critique, but the feedback is a clear signal for future submissions
- The broader discussion highlights a tension between explicitness and concision in commit messages

---

### `git bundle` with bitmaps omits required tree object
**What changed**: Taylor Blau proposed a fix for a bug where `git bundle create` with bitmaps enabled omits tree objects required by advertised refs. The fix restricts pack "haves" to only those UNINTERESTING commits marked as BOUNDARY (i.e., prerequisites recorded in the bundle header).

**Why it matters**: This bug breaks workflows that rely on bundles for offline transport or backup, as recipients cannot access the tree of an advertised commit. The fix ensures all objects required by advertised refs are included in the bundle.

**Technical details**:
- Files touched: `bundle.c`, `t/t6020-bundle-misc.sh`
- The fix aligns the pack-objects traversal with the bundle’s advertised prerequisites
- New test case: `bundle with bitmaps includes trees shared with an excluded sibling`

---

### `git send-email` fails to handle SMTP rate-limiting gracefully
**What changed**: Juha-Matti Tilli reported that `git send-email` fails to handle SMTP rate-limiting errors (e.g., 4.7.1 "too many recipients") gracefully, exiting immediately and leaving remaining patches unsent.

**Why it matters**: This bug causes partial sends and incorrect threading if manually resumed, creating an embarrassing situation for users sending large patch series to high-traffic mailing lists.

**Technical details**:
- Subsystem: `git send-email` (SMTP client logic)
- Error code: SMTP 4.7.1 ("too many recipients")
- Proposed fix: Automatic retry with delay or user prompt for rate-limiting errors
- Workaround: `--batch-size=4 --relogin-delay=60`

---

### MIDX empty layer publication regression fixed
**What changed**: Pia Park posted a v2 patch fixing a regression where MIDX writes publish empty layers (zero objects) without a reverse index, causing subsequent bitmap-enabled writes to fail. The v2 patch expands the fix to cover all MIDX write modes (incremental, non-incremental, and compaction).

**Why it matters**: This regression affects all MIDX write modes and can break bitmap-enabled workflows. The fix prevents publication of invalid layers entirely by exiting early in `write_midx_internal()` when no objects are found.

**Technical details**:
- Files touched: `midx-write.c`, `t/t5334-incremental-multi-pack-index.sh`, `t5326-multi-pack-bitmaps.sh`
- New test cases: empty initial layers, empty packs, and packs with only duplicate objects
- The fix preserves the existing exit status (0) for compatibility with callers

---

## In brief
- **`uploadpack.lazyFetchTrusted` v3**: The series introduces a server-side protected configuration variable for lazy fetching, with a new patch preventing infinite recursion via `GIT_INTERNAL_LAZY_FETCH_DEPTH`. Junio raised concerns about the environment-handling logic and silent-ignoring of non-existent paths.
- **`--force-if-includes` fixes**: Tyler Cipriani posted a v2 series fixing `--force-if-includes` to check the reflog of the pushed ref, not the local branch named after the destination. The series clarifies the detached HEAD rejection policy.
- **ODB refactoring**: Patrick Steinhardt’s ODB refactoring series continues, with Justin Tobler reviewing the deferral of ODB creation in `git clone` and the enforcement of the alternate-only constraint for `git multi-pack-index --object-dir=`.
- **Test modernization**: Junio C Hamano critiqued a test modernization series for not adopting modern test practices ambitiously enough, including proper indentation, logical grouping, and consistent naming conventions.
- **Rust platform support**: Randall S. Becker revealed progress toward Rust platform support (including NonStop), though details are under NDA.
- **`git var` extension**: Junio critiqued the documentation for identity variables as too terse and raised concerns about the output format for multi-valued variables in `-z` mode.
- **`imap-send` OpenSSL fixes**: A three-patch series fixing OpenSSL compatibility and correctness issues in `git imap-send` faces multiple unresolved technical concerns, including a memory-safety bug and incomplete RFC 6125 compliance.