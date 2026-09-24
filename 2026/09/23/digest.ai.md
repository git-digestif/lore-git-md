# Git mailing list daily digest for 2026/09/23

## The day in brief
The Git mailing list saw active discussion on several fronts today. A regression in `git commit --amend` during interactive rebase was slated for a full revert in Git 2.56-rc2, with broader architectural questions deferred. The `http.sslVerifyStatus` feature was reverted from `next` due to test infrastructure issues, while a new `parse-options` sub-API for early argument scanning advanced with a v2 redesign. Performance optimizations for `git ls-files` using the untracked cache were proposed, and several bugfixes—including one for submodule merges with stale delta-base caches—were reported and discussed.

## Notable threads

### Regression in `git commit --amend` during interactive rebase
**What changed**: A regression introduced in Git 2.56-rc0 (commit 6257588252) that prevents `git commit --amend` during interactive rebase conflict resolution will be fully reverted for the 2.56 release cycle. The revert was agreed upon by Junio C Hamano, Elijah Newren, and Patrick Steinhardt after identifying a race condition in the proposed fix and broader architectural concerns about sequencer-based commands.

**Problem**: The original commit aimed to prevent users from accidentally amending the wrong commit during conflict resolution but overreached, blocking legitimate amendments to newly created commits. Elijah Newren identified a critical flaw in the proposed fix: using `MERGE_MSG` as a proxy for conflict state is vulnerable to a race condition with `git reset`, which removes `MERGE_MSG` without advancing `HEAD`.

**Impact**: The revert restores the pre-2.56 behavior, allowing users to amend commits during interactive rebase. The broader architectural question—whether plain `git commit` should be rejected during conflict resolution for sequencer-based commands—is deferred until after Git 2.56. A future direction was clarified: extend `git commit` to consume a new pseudoref (`REBASE_HEAD`), analogous to `CHERRY_PICK_HEAD`, to preserve authorship information during interactive rebase.

**Files touched**: `sequencer.c`, `t/t3404-rebase-interactive.sh` (tests).

**Status**: Revert planned for Git 2.56-rc2.

---

### `http.sslVerifyStatus` reverted from `next`
**What changed**: The `http.sslVerifyStatus` feature, which enables OCSP staple validation for HTTPS connections, was reverted from the `next` branch due to test infrastructure issues reported by SZEDER Gábor. The core functionality remains intact in `master`.

**Problem**: The new `t5585-http-ssl-ocsp.sh` test fails on Gábor’s system due to two issues: (1) the `SSL_VERIFYSTATUS` prerequisite check is too permissive, passing when `git ls-remote` fails for unrelated reasons, and (2) the test’s expectation that a revoked certificate is accepted without `http.sslVerifyStatus` is invalid on systems where libcurl (7.81.0) checks CRLs by default.

**Impact**: The revert is procedural and does not reflect a rejection of the feature. The test suite must be fixed before the feature can graduate to the next release cycle. Gábor also noted two minor code-quality nits in the test script: the test should check the exact error message, and the `with_ssl_verification` helper should use `export` for portability.

**Files touched**: Test infrastructure (`t5585-http-ssl-ocsp.sh`, `t/lib-httpd.sh`).

**Status**: Reverted from `next`; awaiting test suite fixes.

---

### New `parse-options` sub-API for early argument scanning
**What changed**: Christian Couder posted v2 of a series introducing a new `parse-options` sub-API (`early_scan_options()`) to replace fragile hand-rolled early argument scans in Git commands. The v2 redesign addresses Junio C Hamano’s v1 feedback by reusing `struct option` instead of introducing a separate `early_scan_option` structure and focusing on the `git fast-import` bugfix.

**Problem**: The series targets a real bug in `git fast-import` where `--allow-unsafe-features` is silently ignored when preceded by `--depth 5`. The new API is deliberately limited—ignoring short options, negated forms, abbreviations, and subcommands—to prioritize simplicity and speed over completeness.

**Impact**: The v2 series is more aligned with `parse-options` and reduces duplication. Junio raised a substantive usability concern: the API’s requirement for callers to handle argument skipping (`--option value`) could lead to duplication or inconsistencies. The series is now more likely to proceed to `next` once this question is resolved.

**Files touched**: `parse-options.h`, `parse-options.c`, `builtin/fetch.c`, `t/t0040-parse-options.sh`, `t/t9300-fast-import.sh`.

**Status**: Under review; v2 posted.

---

### Performance optimization for `git ls-files` using untracked cache
**What changed**: Tamir Duberstein posted v2 of a three-patch series that teaches `git ls-files` to reuse and update the untracked cache populated by `git status`. The series expands on the original goal of reducing directory traversal overhead in repeated queries.

**Problem**: The series fixes a long-standing inconsistency in ignore-file hash computation (patch 1), enables cache sharing between `git status -unormal` and `git status -uall` (patch 2), and implements cache reuse in `git ls-files` with optional index writes (patch 3).

**Impact**: Benchmarks show dramatic improvements: on a synthetic tree with 100,000 files, a wildcard query (`**/pyproject.toml`) now runs in 33 ms (down from 241 ms upstream). The series also benefits from fsmonitor integration, with similar speedups observed when fsmonitor is enabled.

**Files touched**: `dir.c`, `builtin/ls-files.c`, `t/perf/p3010-ls-files.sh`, `t/t7063-status-untracked-cache.sh`.

**Status**: Under review; v2 posted.

---

### Submodule merge: stale delta-base cache causes corruption
**What changed**: Guillaume Chauvel reported a bug where merging a superproject containing two submodules (A and B) causes Git to attempt to read a commit that exists only in submodule B from submodule A. The merge either fails with "repository corrupt" or misreads the commit due to stale delta-base cache data.

**Problem**: The root cause is a stale cache entry: when a pack is closed, its cached data may linger, and if another submodule’s pack reuses the same memory address and base offset, Git returns incorrect cached data rather than the correct object.

**Impact**: This is a serious bug that can affect users merging superprojects with divergent submodule histories. The author provided a clear, self-contained reproducer that reliably triggers the issue in 93 out of 100 runs across two container environments.

**Files likely touched**: `packfile.c`, `sha1-file.c` (cache invalidation logic).

**Status**: Bug report with reproducer; no patch yet.

---

### `git reflog expire` regression fix
**What changed**: Pushkar Singh posted v2 of a patch fixing a regression in Git 2.50 where the default expiry times for reachable and unreachable reflog entries were swapped. The patch restores the documented behavior: reachable entries expire after 90 days, and unreachable entries after 30 days.

**Problem**: The regression was introduced by commit `85658275702b`, which reversed the order of `.default_expire_total` and `.default_expire_unreachable` in `reflog.h`.

**Impact**: The fix is uncontroversial and narrowly targeted, touching only the misordered defaults in `REFLOG_EXPIRE_OPTIONS_INIT()`. The v2 update addresses Junio’s test-structure concern by consolidating all test cases into a single repository.

**Files touched**: `reflog.h`, `t/t1410-reflog.sh`.

**Status**: Under review; v2 posted.

---

### `git stash` autostash fix for staged index entries
**What changed**: D. Ben Knoble posted v2 of a four-patch series fixing a bug in `git stash` where autostashing fails to correctly handle staged index entries when `stash.index=true` is set. The series replaces subprocess-based index merging with in-core logic via `merge-ort.c`.

**Problem**: The bug manifests during merge operations that trigger autostash, causing incorrect index state preservation due to a race condition in the subprocess-based logic (`git diff-tree` and `git apply`).

**Impact**: The fix eliminates the race condition and simplifies the code. The series is well-structured, with clear precursors (cleanup and refactoring) leading to the functional fix.

**Files touched**: `builtin/stash.c`, `merge-ort.c`, `t/t3900-stash.sh`, `t/t3903-stash.sh`.

**Status**: Under review; v2 posted.

---

### `git-p4` shell injection vulnerability
**What changed**: Anupam Mediratta posted a patch fixing a shell injection vulnerability in `git-p4.py`’s `applyCommit()` function. Junio C Hamano identified a regression in the patch: the new `diffTreeApply()` helper ignores the exit status of the diff-apply pipeline, whereas the original code raised an exception on failure via `p4_system()`.

**Problem**: When a user supplies a commit ID via the `--commit` option, the value is interpolated into a shell pipeline without validation, allowing an attacker to craft a commit ID containing shell metacharacters (e.g., `$(touch evil)`) that will be executed during command substitution.

**Impact**: The patch must be revised to restore the original error-handling semantics while preserving the shell injection mitigation.

**Files touched**: `git-p4.py`.

**Status**: Under review; awaiting revision.

---

### Documentation cross-reference modernization
**What changed**: Julia Evans posted a patch replacing informal plain-text section references (e.g., "see EXAMPLES below") with AsciiDoc link syntax (`<<target,text>>`) across 38 man pages. The patch was approved by Junio C Hamano and queued for integration in `next`.

**Problem**: The goal is to make HTML-rendered man pages more navigable by ensuring all cross-references are clickable, especially when the target section is in an included file.

**Impact**: The patch uses manual anchors (e.g., `[[EDITING_PATCHES]]`) to preserve existing HTML fragment identifiers, avoiding broken external links. Jeff King (Peff) raised a design trade-off: auto-generated anchors (e.g., `_editing_patches`) would align with AsciiDoc toolchain conventions but break external links.

**Files touched**: 38 files in `Documentation/`.

**Status**: Approved and queued for `next`.

---

### CI: Debian 12 HTTP/2 authentication workaround
**What changed**: Johannes Schindelin posted a patch to skip flaky `t5559.15` and `t5559.16` tests in the Debian 12 CI job due to an HTTP/2 authentication failure in curl 7.88.1. Jeff King (Peff) proposed two refinements: (1) a runtime prereq (`HAVE_CURL_HTTP2_BUG`) that detects curl 7.88.1 and skips the affected tests, and (2) a version-range-aware prereq that skips the tests for any curl version between 7.88.1 and 8.3.0 (exclusive).

**Problem**: The original patch is CI-only and hard-codes test numbers, which could silently break if new tests are added earlier in the script. Peff’s alternatives work both in CI and locally and avoid hard-coded test numbers.

**Impact**: The discussion remains focused on which implementation strategy best balances robustness and simplicity. Junio C Hamano critiqued the commit message wording of the version-range approach, calling the "conservative" label misleading.

**Files touched**: `ci/lib.sh` (original patch), `t/t5551-http-fetch-smart.sh` (Peff’s alternatives).

**Status**: Under discussion; no consensus yet.

---

### `git worktree repair` completion support
**What changed**: Yoichi NAKAYAMA posted a patch adding shell completion support for the `git worktree repair` subcommand. The patch was approved by Patrick Steinhardt and is ready for integration.

**Problem**: The completion scripts did not complete the `repair` subcommand for `git-worktree(1)`.

**Impact**: The patch extends the existing path-completion pattern to cover `repair` alongside other `git worktree` subcommands.

**Files touched**: `contrib/completion/git-completion.bash`.

**Status**: Approved and ready for integration.

---

### `git imap-send` OpenSSL compatibility and correctness fixes
**What changed**: Beat Bolli posted a three-patch series fixing correctness and forward-compatibility issues in `git imap-send`’s TLS certificate verification logic. Junio C Hamano identified a memory-safety bug in patch 2, and Patrick Steinhardt proposed an alternative fix using length-bounded string operations.

**Problem**: The series addresses three issues: (1) replacing a deprecated OpenSSL 4.1 ASN1_STRING function, (2) fixing a latent bug where the code incorrectly assumed ASN1_STRINGs are NUL-terminated, and (3) aligning certificate validation with RFC 6125.

**Impact**: The series is under review, with unresolved technical concerns including the memory-safety bug in patch 2 and incomplete RFC 6125 compliance in patch 3.

**Files touched**: `imap-send.c`.

**Status**: Under review; awaiting revision.

---

### `reference-transaction` hook for branch rename/copy
**What changed**: Maciej Ciemborowicz posted v2 of a patch fixing the `reference-transaction` hook to report both deletion and creation events during branch rename (`git branch -m`) or copy (`git branch -c`). The v2 redesign confines copy/rename state to per-ref-update data, preserving transaction genericity and enabling multi-ref operations.

**Problem**: The hook currently only receives the deletion of the old branch name, not the creation of the new one, breaking tools that rely on the hook to monitor ref changes.

**Impact**: The patch is well-motivated and addresses a real pain point for tools like Gerrit and GitLab. The v2 redesign is clean and scalable.

**Files touched**: `refs.c`, `t/t1416-ref-transaction-hooks.sh`.

**Status**: Under review; v2 posted.

---

### `reference-transaction` hook regression fix
**What changed**: Maciej Ciemborowicz posted v5 of a three-patch series fixing a regression where the `reference-transaction` hook receives all-zero object IDs when branches or tags are deleted via high-level commands (`git branch -d`, `git tag -d`, `git remote prune`, `git fetch --prune`). The series was merged to `next`.

**Problem**: The regression was introduced in Git 2.31 and affects both the files and reftable backends. The series restores the pre-2.31 behavior, where the hook receives the resolved old OID followed by all-zeros.

**Impact**: The fix is low-risk and high-impact, addressing a regression that has persisted since Git 2.31. The series includes comprehensive test coverage for SHA-1/SHA-256, broken/symbolic refs, atomic fetch pruning, and concurrent updates.

**Files touched**: `refs.c`, `builtin/fetch.c`, `builtin/remote.c`, `t/t1416-ref-transaction-hooks.sh`.

**Status**: Merged to `next`.

---

### `git rev-parse` shallow history advice
**What changed**: Harald Nordgren posted v3 of a patch adding advice for shallow history in `<rev>~N` and `<rev>^N` syntax. The v3 patch narrows scope to the failing-syntax case, dropping the flawed `git log` integration for silent truncation.

**Problem**: When a user in a shallow clone attempts to access an ancestor not present locally, Git emits a generic "is not a commit" error with no indication that the operation was limited by the shallow boundary.

**Impact**: The new advice (`advice.shallowHistory`) explains the shallow boundary, why the operation failed, and suggests the exact `git fetch --deepen=<n>` command needed to resolve it. The suggested `--deepen` value is context-aware: for `<rev>~N`, it accounts for history already present; for `<rev>^N`, it always suggests `--deepen=1`.

**Files touched**: `advice.c`, `advice.h`, `object-name.c`, `shallow.c`, `shallow.h`, `Documentation/config/advice.adoc`, `t/t1500-rev-parse.sh`.

**Status**: Under review; v3 posted.

---

### CI: GitHub Actions resource exhaustion fixes
**What changed**: Tamir Duberstein posted a two-patch series addressing resource exhaustion in GitHub Actions CI jobs. The series targets two issues: (1) `t4205-log-pretty-formats.sh` generates enormous output that triggers OOM kills when `diff` compares it, and (2) default `make` and `prove` parallelism (`-j10`) is too aggressive for the Linux runner’s 2 CPU cores, causing ENOSPC errors during large clone/repack tests.

**Problem**: The fixes are minimal and pragmatic: (1) replace `test_cmp` with `test_cmp_bin` and delete the large temporary files immediately after comparison, and (2) dynamically set `-j` to the runner’s CPU count.

**Impact**: The series keeps the long-running test cases enabled rather than disabling them, which is a pragmatic choice for CI reliability.

**Files touched**: `.github/workflows/main.yml`, `t/t4205-log-pretty-formats.sh`, `ci/lib.sh`.

**Status**: Under review; v1 posted.

---

### `git config` boolean parsing fix
**What changed**: Colin Hinton posted a patch fixing a bug in `git config` where boolean settings without a value (e.g., `git config --bool core.foo`) would incorrectly return `true` instead of an error. The patch defers the `die(...)` call to the lazy-validation point and addresses the remote-side parser in the same patch.

**Problem**: The current behavior is inconsistent with the documentation, which states that boolean settings must have a value.

**Impact**: The fix ensures that valueless boolean settings are rejected, aligning behavior with documentation.

**Files touched**: `config.c`.

**Status**: Under review; awaiting revision.

---

### "Safe" `strbuf` API RFC
**What changed**: Derrick Stolee posted an RFC series introducing a subset of the `strbuf` API that cannot transitively call `die()` or `exit()`. The series is motivated by the trace2 subsystem, which requires crash-free operation even under memory pressure.

**Problem**: The current `strbuf` API dies on allocation failures, which is incompatible with critical code paths like trace2. The RFC proposes a "safe" API that returns errors instead of dying.

**Impact**: Jeff King (Peff) critiqued the "safe" framing as misleading and overly broad, noting that the API would still not be safe for signal handlers. He proposed an alternative: fixed-size, stack-allocated `strbuf`-like buffers that avoid `malloc()` entirely.

**Files touched**: `strbuf-safe.h`, `strbuf-safe.c`, `strbuf.h`, `strbuf.c`, `wrapper.h`, `wrapper.c`, `json-writer.c`, `tr2_tgt_event.c`, `tr2_tgt_perf.c`.

**Status**: Under discussion; RFC stage.

---

## In brief
- **[PATCH] compat/winansi: fix die_lasterr() formatting bug**: Yongqiang Tian’s patch fixing a long-standing formatting bug in `compat/winansi.c` was acked by Johannes Sixt and is ready for integration.
- **[PATCH] completion: add support for `git worktree repair`**: Yoichi NAKAYAMA’s patch adding shell completion support for `git worktree repair` was approved by Patrick Steinhardt and is ready for integration.
- **[PATCH 0/6] parse-options: new sub-API for early argument scanning**: Christian Couder’s v2 series introducing a new `parse-options` sub-API for early argument scanning is under review, with a substantive usability concern raised by Junio C Hamano.
- **[PATCH 0/4] sequencer: defer auto maintenance until rebase completion**: Thomas Bachem’s series deferring auto maintenance until rebase completion was approved by Phillip Wood and is ready for `next`.
- **[PATCH 0/3] imap-send: OpenSSL compatibility and correctness fixes**: Beat Bolli’s series fixing correctness and forward-compatibility issues in `git imap-send` is under review, with unresolved technical concerns.
- **[PATCH 0/2] ci: reduce resource exhaustion in GitHub Actions**: Tamir Duberstein’s series addressing resource exhaustion in GitHub Actions CI jobs is under review.
- **[PATCH] git-p4: prevent shell injection from user-supplied commit ID**: Anupam Mediratta’s patch fixing a shell injection vulnerability in `git-p4.py` is under review, with a regression identified by Junio C Hamano.
- **[PATCH] Documentation: remove unnecessary comma in `git-rm` man page**: parovozik’s patch removing an unnecessary comma in the `git-rm` man page is under review, with feedback from Junio C Hamano.
- **[PATCH 0/6] repack: fix concurrent-push race leading to data loss**: qeesung’s v3 series fixing a race condition in `git repack -d` is under review, with patch 2/5 acked by Junio C Hamano.
- **[PATCH 0/2] fetch: introduce `+:` refspec for matching-inspired fetch mode**: Harald Nordgren’s series introducing a matching-inspired fetch mode for shallow repositories is under design refinement, with Junio C Hamano proposing a `remote.X.refmap` configuration instead of the `+:` refspec.
- **[PATCH] fetch: HTTP Basic Authentication with empty username**: Xavier Morel reported a bug where Git fails to include the Basic Auth header in follow-up requests when the credential helper supplies an empty username with a non-empty password.