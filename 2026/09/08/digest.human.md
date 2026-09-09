# Git mailing list daily digest for 2026/09/08

## The day in brief
The `git history squash` feature reached a major milestone with its v15 reroll, now technically complete and ready for integration. Meanwhile, a critical design flaw was identified in the `receive-report` hook series, blocking its graduation from `next`. The `uploadpack.lazyFetchTrusted` series saw new reviews questioning its environment-handling logic and URI error messages, while a regression in `git bundle` with bitmaps was reported and promptly fixed.

## Notable threads

### `git history squash` v15: feature-complete and ready for integration
**What changed?**
Harald Nordgren’s v15 reroll of the `git history squash` feature enforces strict case-sensitive matching for autosquash markers (e.g., rejecting `fixup! ABCDEF` while accepting `fixup! abcdef`). This aligns with Git’s historical convention of emitting only lowercase hexadecimal OIDs, as established by `bc/restrict-hex-to-lowercase`.

### Why it matters

The series is now technically complete, addressing all prior feedback on autosquash marker resolution, shape-based validation, ref protection for local branches, and the `--no-edit` workflow. Junio C Hamano’s "Will replace" sign-off from v7 signals intent to queue it for the next release. The feature collapses a commit range into its oldest ancestor while preserving descendant history, avoiding the repeated conflict stops of a rebase-based approach.

### Key details

- **Files touched**: `builtin/history.c`, `sequencer.c`/`sequencer.h`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`, `t/t9902-completion.sh`.
- **New helpers**: `first_parent_tree_oid()`, `resolve_squash_range()`, `squash_check_autosquash_subject()`, `build_squash_message()`.
- **Behavior**: Rejects ranges that are empty, single-commit, contain root commits, have multiple tips, or include merges with external parents. Ref protection blocks operations if any local branch descended from the squashed range would be left dangling.

---

### `receive-report` hook: critical design flaw blocks graduation
**What changed?**
Junio C Hamano identified a critical design flaw in the `receive-report` hook series: the `BUG()` call in the `switch` statement assumes the client’s requested protocol version is always valid, but clients may omit the `report-status` or `report-status-v2` capability entirely, leaving `version` as `REPORT_STATUS_UNKNOWN`.

### Why it matters

The hook is queued in `next` but cannot graduate to `master` until this flaw is fixed. The series introduces a server-side filter for the status report sent to clients after ref updates, enabling GitLab’s MVCC use case. The intentional decoupling between reported status and actual repository state violates Git’s traditional consistency guarantees, making this a high-stakes design decision.

### Key details

- **Files touched**: `builtin/receive-pack.c`, `Documentation/githooks.adoc`, `Documentation/git-receive-pack.adoc`, `t/t5412-receive-report-hook.sh`.
- **Fix required**: Replace `default: BUG(...)` with an explicit `REPORT_STATUS_UNKNOWN` case that `break`s, ensuring the code silently skips the status report when the client does not request one.
- **Test coverage**: 224 lines in `t/t5412-receive-report-hook.sh` covering passthrough, failure modes, stdin/stdout handling, and sideband relay.

---

### `uploadpack.lazyFetchTrusted`: environment-handling and URI error messages
**What changed?**
Junio C Hamano and Kaartic Sivaraam raised new concerns about the `uploadpack.lazyFetchTrusted` series. Junio critiqued the environment-handling logic in patch 5/5, suggesting a cleaner approach to preserve operator control. Kaartic noted that error messages now print full URIs (e.g., `garbage://hello-world`) instead of just the format name (e.g., `garbage`), which could reduce clarity.

### Why it matters

The series introduces a server-side protected configuration variable to mark repositories as trusted for lazy fetching, addressing security concerns around lazy fetching in promisor-remote workflows. The unresolved feedback may delay integration, though the core design remains uncontroversial.

### Key details

- **Files touched**: `promisor-remote.c`, `setup.c`, `upload-pack.c`, `builtin/upload-pack.c`, `Documentation/config/uploadpack.adoc`, `t/t5710-promisor-remote-capability.sh`.
- **New symbols**: `path_allowlist_apply()`, `upload_pack_lazy_fetch_trusted()`, `GIT_INTERNAL_LAZY_FETCH_DEPTH`.
- **Behavior**: `uploadpack.lazyFetchTrusted` allows server operators to mark repositories as trusted for lazy fetching, with recursion safeguards preventing infinite loops.

---

### `git bundle` with bitmaps: regression fixed
**What changed?**
Taylor Blau fixed a regression where `git bundle create` with bitmaps enabled omitted tree objects required by advertised refs. The fix restricts pack "haves" to only those UNINTERESTING commits marked as BOUNDARY (i.e., prerequisites recorded in the bundle header).

### Why it matters

The bug caused bundles to pass verification and unbundle successfully but fail when recipients tried to access the tree of an advertised commit. This affected workflows relying on bundles for offline transport or backup.

### Key details

- **Files touched**: `bundle.c`, `t/t6020-bundle-misc.sh`.
- **Test coverage**: New test case (`bundle with bitmaps includes trees shared with an excluded sibling`) verifies the fix.
- **Behavior**: The fix aligns the pack-objects traversal with the bundle’s advertised prerequisites, ensuring all objects required by advertised refs are included.

---

### `--force-if-includes` fixes: reflog timestamp regression resolved
**What changed?**
Tyler Cipriani’s v2 series clarifies that `--force-if-includes` already rejects detached HEAD pushes today (when the same-named local branch lacks the remote tip), and this change makes that rejection policy more transparent. The fix ensures the reflog of the pushed ref is checked, not the local branch named after the destination.

### Why it matters

The original behavior could cause false rejections or unintended data loss when pushing from a differently named local branch or detached HEAD. The v2 series addresses a regression in the reflog timestamp check and improves detached HEAD advice.

### Key details

- **Files touched**: `remote.c`, `advice.c`, `advice.h`, `builtin/push.c`, `send-pack.c`, `transport-helper.c`, `transport.c`, `t/t5533-push-cas.sh`.
- **New symbols**: `advice.forceIfIncludesDetachedHead`, `ADVICE_PUSH_REF_UNVERIFIABLE`, `ref->unverifiable`.
- **Test coverage**: Eight new test cases covering mismatched local/remote names, `HEAD`-based pushes, and detached HEAD states.

---

## In brief
- **`git var` extension**: Junio C Hamano critiqued the documentation for identity variables (`GIT_AUTHOR_NAME`, etc.) as too terse and raised concerns about the output format for multi-valued variables in `-z` mode.
- **ODB refactoring**: Justin Tobler reviewed Patrick Steinhardt’s ODB alternates series, suggesting tighter conditions for local clones and confirming the fix for a latent bug in worktree alternates handling.
- **Auto maintenance deferral**: Junio C Hamano signaled intent to queue Thomas Bachem’s series, which defers auto maintenance until sequencer-driven operations complete, but critiqued modern commit messages for verbosity.
- **Ref storage format**: Kaartic Sivaraam raised procedural questions about the backward-compatibility strategy for the old `--ref-format=` option and noted inconsistencies in the `PARSE_OPT_NONEG` flag.
- **Git 3.0 readiness**: Randall S. Becker updated the community on Rust platform support, revealing progress toward compatibility on platforms like NonStop, though details remain under NDA.
- **Test modernization**: Junio C Hamano critiqued Mark C. Chu-Carroll’s test modernization series for not adopting modern test practices ambitiously enough, suggesting deeper structural improvements.
- **`imap-send` fixes**: Junio C Hamano and Patrick Steinhardt identified memory-safety and design issues in Beat Bolli’s `imap-send` series, including a heap corruption bug and premature OpenSSL 4.1 compatibility macro.
- **`git bundle` bug**: Peter Elmers reported a bug where `git bundle create` with bitmaps omits required tree objects, which Taylor Blau promptly fixed.
- **`git send-email` rate-limiting**: Juha-Matti Tilli reported that `git send-email` fails to handle SMTP rate-limiting errors gracefully, exiting immediately and leaving remaining patches unsent.