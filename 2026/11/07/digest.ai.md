# Git mailing list daily digest for 2026/11/07

## The day in brief
The Git mailing list saw significant activity today, with **design objections** raised on the `git repo info` path keys series, a **UI/UX refinement proposal** for `git format-rev`, and **critical bugfixes** for `git apply --check` crashes and `split-index`/`skipHash` corruption. Several patches were marked ready for `next`, including fixes for sideband control-character parsing and `the_repository` removal in `merge-blobs`. A 15-patch series addressing segfaults on unvalued config keys was posted, covering 17 crash sites across multiple commands.

## Notable threads

### [PATCH v8] git repo info: path-related keys
**What changed**: Phillip Wood raised **high-weight design objections** to the naming and necessity of several new path-related keys in `git repo info`. The objections target:
- Naming: `path.toplevel` (→ `path.working-tree`), `path.superproject-root` (→ `path.superproject-working-tree`), and `path.git-prefix` (→ `path.prefix`).
- Necessity: `path.hooks` is argued to be unnecessary, as scripts should use `git hook run` or config-based hooks.
- Behavioral consistency: `path.cdup` may unexpectedly dump users in their home directory when run from the repository root, mirroring `git rev-parse --cdup`.

**Why it matters**: The series is mechanically complete and ready for merging, but these objections could require significant revisions or maintainer arbitration. The risk of abandonment due to lack of reviewer engagement remains critical.

**Subsystem**: `git repo info` (scripting interface).
**Kind of change**: Feature (GSoC project).
**Impact**: High (design decisions may affect usability and long-term maintainability).

---

### [PATCH 0/5] format-rev: add --abbrev, --color, --date options
**What changed**: D. Ben Knoble proposed a **UI/UX refinement** to split `git format-rev` into two commands:
- Move `--stdin-mode=revs` (formatting revision ranges) to `git diff-tree` (which would need to learn NUL-terminated output).
- Retain `git format-rev` exclusively for `--stdin-mode=text` (turning hashes into references in arbitrary input text).

**Why it matters**: The proposal aims to simplify invocation (e.g., for editor integration) and reduce the command’s scope. The `--stdin-mode=text` mode’s unique functionality justifies its existence, but the `--stdin-mode=revs` mode’s necessity remains debated. This could influence the series’ future direction or motivate a follow-up effort to modify `git diff-tree`.

**Subsystem**: `git format-rev` (experimental command).
**Kind of change**: UI/UX refinement (conceptual).
**Impact**: Medium (could reshape the command’s design and adoption).

---

### [PATCH] merge-ort: fix --no-overwrite-ignore regression
**What changed**: Aniket Gupta posted a **bugfix** for the `--no-overwrite-ignore` regression in `merge-ort`, where ignored files were overwritten during directory rename detection. The fix introduces a `preserve_ignored` bit in `struct merge_options` and updates `check_preimage()` to reject ignored files.

**Why it matters**: The regression violates the documented contract of `--no-overwrite-ignore` and risks data loss for users relying on it to protect local configuration, credentials, or build artifacts. The fix is minimal and well-motivated, addressing a high-impact issue in the `merge-ort` strategy.

**Subsystem**: `merge-ort` (merge strategy).
**Kind of change**: Bugfix (regression).
**Impact**: High (data loss prevention).

---

### [PATCH 1/2] unpack-trees: remove USE_THE_REPOSITORY macro
**What changed**: Junio C Hamano posted a **refactoring series** removing the `USE_THE_REPOSITORY` macro from `unpack-trees.c` as part of the ongoing `the_repository` removal effort. The series introduces a new `unpack_repo()` helper and standardizes variable naming.

**Why it matters**: This is a mechanical step in the project-wide effort to eliminate implicit repository references, improving code clarity and maintainability. The series is uncontroversial and ready for integration.

**Subsystem**: `unpack-trees.c` (index handling).
**Kind of change**: Refactoring.
**Impact**: Low (internal cleanup).

---

### [PATCH 0/15] Fix segfaults on unvalued config keys
**What changed**: Thomas Bachem posted a **15-patch bugfix series** addressing segfaults when config settings are given without values (e.g., `git -c foo.bar`). The series audits config callbacks and `repo_config_get_value()` callers, fixing 17 crash sites across commands like `git rebase -i`, `git fetch`, `git add -p`, and `git clone --bundle-uri`.

**Why it matters**: The series systematically eliminates a class of latent crashes, improving Git’s robustness. One patch (15/15) fixes a regression introduced in Git 2.54.0. The audit’s completeness suggests it is likely to catch most remaining instances of this bug class.

**Subsystem**: Config parsing (multiple commands).
**Kind of change**: Bugfix (systemic).
**Impact**: High (crash prevention).

---

### split-index: fix corruption with skipHash and splitIndex enabled
**What changed**: Tanga Joris **diagnosed the root cause** of a critical bug in the index subsystem: `write_shared_index()` names the shared index file after its trailing checksum, which is written as zeros when `index.skipHash` is enabled. This causes a null base OID, leading to silent data loss and crashes in `git status`.

**Why it matters**: The bug affects users of `core.splitIndex` and `index.skipHash` (e.g., large repositories with `feature.manyFiles`), causing index corruption and crashes. The diagnosis enables a targeted fix, which Tanga is actively working on.

**Subsystem**: Index handling (`split-index.c`, `wt-status.c`).
**Kind of change**: Bugfix (critical).
**Impact**: High (data loss and crashes).

---

### [PATCH 0/2] git history: add --verbose support for reword
**What changed**: D. Ben Knoble **questioned the choice of `--verbose`** for `git history reword`, suggesting alternatives like `--commit-verbose` or `--diff-in-editor` to avoid overloading the term in the context of `git history`. The author, Jelle van der Waa, proposed the feature to show the full diff while editing the commit message, mirroring `git commit --amend --verbose`.

**Why it matters**: The feedback highlights a potential usability issue: "verbose" in `git history` could imply a broader range of behaviors (e.g., post-execution summary of rewritten commits). The discussion may prompt a reconsideration of the flag name before the feature solidifies.

**Subsystem**: `git history` (new built-in command).
**Kind of change**: Feature (UI/UX).
**Impact**: Medium (user interface clarity).

---

## In brief
- **[PATCH] sideband: fix infinite loop in control-character config parser**: Junio C Hamano marked the fix as ready for `next`. The patch advances the parser to the next comma-separated entry after warning about an unrecognized keyword, preventing infinite loops on malformed configurations.
- **[PATCH] merge-blobs: remove implicit dependency on the_repository**: Junio C Hamano marked the refactoring patch as ready for `next`, removing the `USE_THE_REPOSITORY` macro from `merge-blobs.c`.
- **[PATCH 1/2] git apply --check: fix crash on malformed patches**: Tanga Joris posted a two-part fix for crashes in `git apply --check` when processing patches with stale `old_name`/`new_name` values or inconsistent header ordering.
- **[PATCH] apply: fix regression in pure-rename patch series**: Aniket Gupta confirmed the regression scenario (pure rename followed by edit) is valid input produced by `git format-patch`, resolving Junio C Hamano’s technical concerns.
- **[PATCH 0/5] http: exponential backoff, configurable retry codes, POST retries**: Junio C Hamano questioned the two-phase default value introduction for `http.retryAfter` (patches 1–4 assume 0, patch 5 sets 10s) and solicited brian m. carlson’s input on maintainability.
- **[PATCH] hash: prevent panic when formatting ObjectID with invalid algorithm**: Tanga Joris requested the patch author incorporate brian m. carlson’s commit to make the `algo` field non-public, preventing invalid states.
- **[PATCH] partial-clone: clarify fetch ordering for promisor remotes**: Junio C Hamano asked whether the documented behavior is long-standing or fills a gap left by commit `8808e61fd3`, and requested Christian Couder’s input on coordination of recent promisor-remote patches.
- **[PATCH 0/3] test-tool genbytes: replace slow tr(1) invocations**: Junio C Hamano proposed a hexadecimal byte-specifier interface for `genbytes` (e.g., `00` for NUL) instead of accepting "NUL" as a string, and suggested squashing patches 1 and 2 to avoid breaking tests mid-series.