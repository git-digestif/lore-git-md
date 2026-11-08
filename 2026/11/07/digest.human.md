# Git mailing list daily digest for 2026/11/07

## The day in brief
Phillip Wood raised **design objections** to the `git repo info` path keys series, questioning naming, necessity, and edge-case behavior. A **critical index corruption bug** in `splitIndex`/`skipHash` was diagnosed, with a fix in progress. Junio C Hamano **marked three topics ready for `next`**: sideband control-character parser, `merge-blobs` repository refactoring, and i18n cleanup for `git describe`. A **15-patch series** fixed segfaults on unvalued config keys across 17 crash sites.

## Notable threads

### [PATCH v8] git repo info: new path-related keys
**What changed**: Phillip Wood raised high-weight design objections to the mechanically complete v8 series adding seven new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, `path.cdup`) to `git repo info`.

**Problem/goal**: Expose filesystem locations of repository components in a scriptable format, addressing prior design objections by adding `path.cdup` for symmetry with `path.git-prefix`.

**Subsystem**: `git repo info` command (GSoC project).

**Impact**: The series is at risk of abandonment due to lack of reviewer engagement, but Phillip’s objections—naming (`path.toplevel` → `path.working-tree`, `path.superproject-root` → `path.superproject-working-tree`, `path.git-prefix` → `path.prefix`), necessity of `path.hooks`, and behavioral consistency of `path.cdup`—must be resolved before merging.

**Today’s delta**: Phillip Wood’s review [2026/11/07/19-57-02] directly engages with the core design, questioning whether the keys are necessary or well-named. The `path.hooks` objection is fundamental: Phillip argues scripts should use `git hook run` or config-based hooks instead of querying the hooks directory directly. The `path.cdup` edge case (unexpected behavior at repository root) could affect script portability.

---

### split-index: fix corruption with skipHash and splitIndex enabled
**What changed**: Tanga Joris diagnosed the root cause of a critical bug causing silent index corruption (null OIDs, missing entries) and crashes (`BUG: wt-status.c:585`) when both `core.splitIndex` and `index.skipHash` are enabled.

**Problem/goal**: Fix index corruption and crashes in repositories using split-index or skipHash optimizations (e.g., large repos with `feature.manyFiles`).

**Subsystem**: Index file handling (`split-index.c`, `wt-status.c`).

**Impact**: The bug blocks users of split-index or skipHash optimizations, causing data loss and crashes during `git status`. The root cause is `write_shared_index()` naming the shared index file after its trailing checksum, which is written as zeros when `index.skipHash` is enabled, resulting in a null base OID and silent data loss.

**Today’s delta**: Tanga Joris’s diagnosis [2026/11/07/21-10-36] traces the bug to `write_shared_index()` and explains how the null base OID leads to data loss. A fix is in progress, with no maintainer feedback yet.

---

### [PATCH] segfaults: fix unvalued config keys
**What changed**: Thomas Bachem posted a **15-patch series** fixing segfaults when config settings are given without values (e.g., `git -c foo.bar` or bare `bar` under `[foo]`), covering 17 crash sites across commands like `git rebase -i`, `git fetch`, `git add -p`, and `git clone --bundle-uri`.

**Problem/goal**: Systematically eliminate crashes triggered by unvalued config keys, hardening Git’s config parsing against NULL dereferences.

**Subsystem**: Config parsing (`config.c`, `repo_config_get_value()` callers).

**Impact**: The series addresses a systemic issue affecting 17 distinct crash sites, including a server-triggered crash via bundle lists (patch 14/15) and a regression in `git config list --type=color/expiry-date` (patch 15/15). Most fixes reject missing values with `config_error_nonbool()`, following the precedent set by ba176db511 (2023-12-07). Three exceptions handle the absence gracefully: `fetch.bundleCreationToken` defaults to 0, remote URLs skip `includeIf` matching, and `git config list` omits the key entirely.

**Today’s delta**: The series [2026/11/07/08-42-32] is based on `master` and merges cleanly, though it conflicts with two topics in `seen` (`iw/connected-deepen-fetch-no-alternate-refs` and `kk/quarantine-alternate-refs`) due to overlapping test additions. No reviews yet.

---

### [PATCH 0/5] http: exponential backoff, configurable retry codes, POST retries
**What changed**: Ravi Mistry posted v3 of a five-patch series addressing gaps in Git’s HTTP transport error handling, incorporating the 10-second `http.retryAfter` default and adding patch 5/5 to formalize it.

**Problem/goal**: Improve resilience when interacting with hosting backends and reverse proxies under high concurrency by fixing an off-by-one retry count bug, introducing exponential backoff, making retryable status codes configurable, and extending retries to HTTP POST requests.

**Subsystem**: HTTP transport (`http.c`, `remote-curl.c`).

**Impact**: The series fixes a measurable performance issue (tight retry loops) and extends retry logic to POST requests, which carry the bulk of Git’s data transfer. The 10-second default strikes a balance between server protection and interactive usability. Junio C Hamano raised a **procedural concern** [2026/11/07/04-37-59] about the two-phase default value introduction (patches 1–4 assume 0, patch 5 sets 10s), questioning whether it is the right way to structure the series for maintainability.

**Today’s delta**: The v3 series [2026/11/07/03-40-31] is technically complete and awaits Junio’s final review. The 10-second default is uncontested, but the patch organization may need adjustment.

---

### [RFC PATCH 0/2] git history: add --verbose support for reword
**What changed**: D. Ben Knoble questioned the choice of `--verbose` for `git history reword`, suggesting alternatives like `--commit-verbose` or `--diff-in-editor` to avoid overloading the term in the context of `git history`.

**Problem/goal**: Add `--verbose` support to `git history reword` to show the full diff while editing the commit message, mirroring `git commit --amend --verbose`.

**Subsystem**: `git history` (new built-in command).

**Impact**: The feature is user-facing and aligns with existing Git conventions, but Knoble’s review [2026/11/07/19-49-53] raises a conceptual concern: "verbose" in `git history` could imply a broader range of behaviors (e.g., post-execution summary of rewritten commits), and alternatives like `--commit-verbose` or `--diff-in-editor` might better clarify the behavior.

**Today’s delta**: Knoble’s feedback may prompt a redesign of the flag naming before the feature solidifies.

---

### [PATCH] partial-clone: clarify fetch ordering for promisor remotes
**What changed**: Junio C Hamano asked whether the documented fetch ordering behavior for promisor remotes reflects long-standing behavior or fills a gap left by commit `8808e61fd3` (April 2026), and requested Christian Couder’s input on coordination of recent promisor-remote patches.

**Problem/goal**: Clarify the fetch ordering behavior for promisor remotes in `Documentation/technical/partial-clone.adoc`.

**Subsystem**: Partial-clone (promisor remotes).

**Impact**: The patch is a documentation update with no code changes, but Junio’s review [2026/11/07/22-55-29] raises a **coordination concern**: if multiple contributors are working on promisor remotes without alignment, the risk of conflicting or redundant patches increases. Christian Couder is CC’d as the domain expert.

**Today’s delta**: Junio’s questions about the patch’s motivation and coordination remain unanswered.

---

## In brief
- **[PATCH] merge-ort: fix --no-overwrite-ignore regression**: Aniket Gupta posted a fix [2026/11/07/04-44-19] for the `--no-overwrite-ignore` regression in `merge-ort`, adding a `preserve_ignored` bit to `struct merge_options` and updating `check_preimage()` to reject ignored files.
- **[PATCH] apply: fix regression in pure-rename patch series**: Aniket Gupta confirmed [2026/11/07/04-07-15] the regression scenario (pure rename followed by edit) is valid input produced by `git format-patch`, not corrupt data, and clarified [2026/11/07/05-47-01] the bug reproduces in all three workflows: single `git apply`, separate invocations, and `git am`.
- **[PATCH] sideband: fix infinite loop in control-character config parser**: Junio C Hamano marked the fix [2026/11/07/02-20-36] ready for `next`.
- **[PATCH] merge-blobs: remove implicit dependency on the_repository**: Junio C Hamano marked the `the_repository` removal patch [2026/11/07/02-30-15] ready for `next`.
- **[PATCH] git apply --check: fix crashes on malformed patches**: Tanga Joris posted a two-part fix [2026/11/07/19-54-28] for the `git apply --check` crash, addressing stale `old_name`/`new_name` values (1/2) and inconsistent header ordering (2/2).
- **[PATCH] unpack-trees: remove USE_THE_REPOSITORY macro**: Junio C Hamano posted a two-patch series [2026/11/07/01-26-52] removing `USE_THE_REPOSITORY` from `unpack-trees.c`, introducing `unpack_repo()` and renaming local variables for consistency.
- **[PATCH] test-tool genbytes: replace slow tr(1) invocations**: Junio C Hamano suggested [2026/11/07/23-15-09] squashing patches 1 and 2 of the `test-tool genbytes` series to avoid breaking tests mid-series, and proposed [2026/11/07/23-23-25] a hexadecimal byte-specifier interface for `genbytes` (e.g., `00` for NUL) instead of accepting "NUL" as a string.