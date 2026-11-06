# Git mailing list daily digest for 2026/11/05

## The day in brief
The Git mailing list saw significant activity around performance optimizations, bug fixes, and documentation improvements. Key developments include Junio Hamano's approval of a fix for `git annotate` completion, a critical bug report about commit-graph corruption in Git 2.54's geometric maintenance strategy, and ongoing discussions about HTTP retry improvements and promisor remote ordering. Outreachy proposals for libification and argument parsing improvements also generated interest.

## Notable threads

### [RFC PATCH 0/5] Add merge commit support to "git history"
**What changed**: Philippe Blain identified a regression in boundary commit handling when replaying merges with negative revisions (e.g., `M^!`). The pre-seeding logic incorrectly maps a merge's second parent to the `--onto` target, altering the merge base and causing unnecessary conflicts.

**Problem/goal**: The series aims to extend "git history" to handle merge commits, which were previously unsupported. The regression affects core use cases like replaying merges with unchanged trees.

**Subsystem**: `replay.c`, merge composition logic.

**Change type**: Bugfix (regression).

**Impact**: High for users of `git replay` with merge commits. The bug could block adoption if unresolved.

**Today's news**: Blain provided a concrete reproduction scenario and suggested actionable fixes: adjust the pre-seeding logic to exclude negative revisions that aren't boundary commits, and document the behavior. This is a high-signal review that directly addresses a core use case.

---

### [PATCH 0/15] push: speed up refspec matching for many deletions
**What changed**: Junio Hamano reviewed multiple patches in Jon Simons' performance series, approving several optimizations. The series achieves 10–100x speedups in `git push` refspec matching by replacing linear traversals with `strmap` lookups.

**Problem/goal**: Client-side refspec matching becomes a bottleneck when pushing many branch deletions to servers with large numbers of refs (e.g., 100k). The series fixes this and addresses correctness gaps in refspec ordering and duplicate-destination handling.

**Subsystem**: `refs.c`, `remote.c`, `builtin/send-pack.c`.

**Change type**: Performance optimization and bugfix.

**Impact**: High for users pushing many deletions to large repositories. The optimizations are significant (61–79% speedups in benchmarked scenarios).

### Today's news

- Junio approved the third patch (eliminating `mkpath()` overhead) but asked for clarification about nested path matching in the new `match_parse_rule()` helper.
- Junio approved the eighth patch (client-side rejection of duplicate destinations) and the twelfth patch (optimizing `--force-with-lease` overhead), calling the latter "clever and apparently very effective."
- The series is now largely approved, with only minor clarifications needed.

---

### [PATCH] fsmonitor: invalidate stale untracked cache directories after trivial response
**What changed**: Derrick Stolee withdrew his competing patch in favor of Ilia Kurenkov's approach, which preserves parts of the untracked cache rather than invalidating it entirely. Stolee endorsed Ilia's patch as both correct and more performant.

**Problem/goal**: Fix a long-standing bug where the untracked cache remains incorrectly valid after fsmonitor fails or returns a trivial response, causing new untracked files to be invisible to `git status`.

**Subsystem**: fsmonitor, untracked cache.

**Change type**: Bugfix.

**Impact**: High for users of fsmonitor in large repositories. The bug affects core functionality (`git status` visibility of untracked files).

**Today's news**: Stolee's withdrawal and endorsement of Ilia's patch resolve the procedural question about which patch to merge. The path forward is now clear, though Ilia's patch still awaits formal review.

---

### [PATCH] odb.c: skip self-referential alternates during quarantine
**What changed**: Junio Hamano confirmed the patch is ready for integration after Jeff King's explicit endorsement. The patch fixes a performance regression in `git-receive-pack` connectivity checks by skipping listing own refs as alternates during quarantine.

**Problem/goal**: The quarantine mechanism unnecessarily loads all refs—including hidden ones—from the repository's own object directory, causing a measurable performance regression (65.5% speedup with 200,000 hidden refs).

**Subsystem**: ODB abstraction, quarantine plumbing.

**Change type**: Performance optimization and bugfix.

**Impact**: High for users with large numbers of hidden refs. The fix is minimal (10 lines) and future-proof.

**Today's news**: Junio's confirmation closes the technical discussion, marking the patch as ready for `next`.

---

### [PATCH] hash: prevent panic when formatting ObjectID with invalid algorithm
**What changed**: Tanga Joris identified a potential inconsistency with the C codebase: the Rust implementation treats algorithm ID 0 as invalid (appending `:invalid-hash-algo-0`), while the C code silently falls back to the default hash algorithm.

**Problem/goal**: Fix a panic in Git's Rust `hash` module when formatting an `ObjectID` with an unrecognized algorithm identifier.

**Subsystem**: Rustification (hash module).

**Change type**: Bugfix.

**Impact**: Medium for Rust-based Git tools. The inconsistency could cause divergent behavior between the two codebases.

**Today's news**: The review highlights a discrepancy that may require resolution before merging.

---

### [PATCH] rev-list: fix `--grep` and related option parsing when followed by another option
**What changed**: Azuama Odinaka Maximus expanded the fix to cover all 18 `parse_long_opt()`-style options (e.g., `--grep`, `--author`, `--since`) that previously misparsed their arguments when followed by another option-like string.

**Problem/goal**: The `git rev-list` command misparses options like `--grep` when the option is immediately followed by another argument that begins with `--`.

**Subsystem**: `builtin/rev-list.c`.

**Change type**: Bugfix.

**Impact**: Medium for users of `git rev-list` with complex option combinations. The fix ensures consistent argument parsing.

**Today's news**: The v4 patch addresses all prior feedback, including test indentation and documentation updates. The expanded scope makes the fix more comprehensive.

---

### [PATCH] path-walk: fix object loss when tagged tree revisits path
**What changed**: Junio Hamano acknowledged Taylor Blau's "LGTM" for Derrick Stolee's patch, which fixes a regression in Git 2.55.0 where the pack-objects/path-walk machinery drops objects when a tagged tree contains a blob at a path already visited by a prior tree batch.

**Problem/goal**: Fix a regression where objects at paths already in `path_stack_pushed` were silently dropped, causing missing objects in packfiles.

**Subsystem**: pack-objects/path-walk.

**Change type**: Bugfix (regression).

**Impact**: High for users of `git pack-objects` with tagged trees. The bug affects core functionality.

**Today's news**: The patch is now effectively approved for merging, though it still lacks substantive review from domain experts.

---

### [PATCH] promisor-remote: add promisor.fetchOrder config
**What changed**: Yopa Nelly posted a v2 patch addressing all technical feedback from Tanga Joris's review. The patch now includes documentation and avoids the file-static `fetch_order` list that caused architectural issues in v1.

**Problem/goal**: Allow users to specify the order in which Git tries multiple promisor remotes during lazy fetches, replacing the current implicit order (configuration file order).

**Subsystem**: promisor-remote.

**Change type**: Feature.

**Impact**: Medium for users of partial clones with multiple promisor remotes. The feature addresses a concrete interoperability gap.

**Today's news**: The v2 patch resolves the crash and architectural flaws identified in v1. Junio Hamano asked Christian Couder how the patch meshes with server-side promisor remote advertisement.

---

### [PATCH] builtin/annotate.c: fix command-line completion for `git annotate`
**What changed**: Junio Hamano approved Tanga Joris's patch, which fixes a bug where `git annotate --<TAB>` incorrectly offered `git diff` options instead of `git annotate`'s own.

**Problem/goal**: Fix command-line completion for `git annotate` by removing redundant `-c` prepending in `cmd_annotate()`.

**Subsystem**: `builtin/annotate.c`.

**Change type**: Bugfix.

**Impact**: Low for most users, but improves the CLI experience for `git annotate`.

**Today's news**: Junio's approval signals the patch is ready for `next`.

---

### Bug report: geometric maintenance strategy causes commit-graph corruption
**What changed**: Christoph Weiss reported a bug in Git 2.54's geometric maintenance strategy where `git maintenance run --auto` prunes unreachable commits but fails to update the commit-graph, leaving it in an inconsistent state.

**Problem/goal**: The `geometric` strategy (now the default) causes commit-graph corruption after pruning unreachable commits, affecting operations that rely on the commit-graph.

**Subsystem**: commit-graph, maintenance.

**Change type**: Bug report (regression).

**Impact**: High for users upgrading to Git 2.54. The bug affects core functionality and could impact performance and correctness.

**Today's news**: This is the first report of the issue. The reproduction script is self-contained and demonstrates the corruption via `git fsck`.

---

## In brief
- **[PATCH v2] credential-cache: fix race condition in daemon socket handling**: Junio Hamano asked for a technical justification or revised implementation regarding the architectural concern about mixing `unix_ss_*` and `tempfile` APIs.
- **[PATCH v3] bundle-uri: handle relative URIs with excessive parent traversals**: Aniket Gupta posted v3 with improved commit messages and test tightening, addressing Patrick Steinhardt's feedback.
- **[PATCH] environment: move `pack.packsizelimit` into `repo_config_values`**: Tanga Joris identified a procedural blocker: Junio's `jc/freeze-repo-config-values` topic prohibits adding new members to `struct repo_config_values`.
- **[PATCH] completion: remove NO_PARSEOPT for commands using parse-options**: Junio Hamano suggested rewording a comment in `parse-options.h` to focus on the function's general purpose rather than over-specifying the `fast-import` edge case.
- **[PATCH] diff: rewrite EXAMPLES section of git-diff(1)**: Junio Hamano posted a squashable fix-up addressing mechanical oversights in the `--staged`/`--cached` presentation.
- **[PATCH] http: exponential backoff, configurable retry codes, POST retries**: Brian M. Carlson rejected Ravi Mistry's compromise proposal (`http.retryAfter=1`) as insufficiently protective of rate-limited servers.
- **[PATCH] CodingGuidelines: update minimum Perl version**: Junio Hamano queued the patch updating `Documentation/CodingGuidelines` to reflect the bump in Git's minimum required Perl version from 5.8.1 to 5.26.0.
- **[PATCH] api-parse-options.adoc: document missing flags and macros**: Karthik Nayak acknowledged all review feedback and plans to address copy-editing nits and commit-message inaccuracies in v2.
- **[PATCH] ls-remote: add --show-object-format option**: Adrian Ratiu acknowledged Patrick Steinhardt's feedback and plans to address it in v2, including prefixing the output and parameterizing the test helper.
- **[PATCH 0/5] object and ref storage test suite preparation**: Patrick Steinhardt posted a five-patch series preparing the Git test suite to run with non-default object storage backends.
- **[Outreachy] Configurable fetch ordering for promisor remotes in partial clones**: Shashank S Mogaveera revised the proposal to address Kaartic Sivaraam's feedback, deferring config format and server-side advertisement mechanism to RFCs.
- **[Outreachy] Improving Git command argument scanning and parsing**: Tanga Joris proposed improving consistency and robustness in how Git built-in commands scan and parse command-line arguments, particularly for early requests like `-h` and `--git-completion-helper`.