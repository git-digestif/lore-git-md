# Here is the concise, contextual digest for 2026/10/26:

---

# Git mailing list daily digest for 2026/10/26

## The day in brief
The list saw **179 emails** across **60 threads**, with key developments in **macOS CRLF conversion**, **repack data-loss fixes**, **CI tooling**, and **Git libification**. The most consequential new deltas were **Torsten Bögershausen’s RFC optimization** for the macOS CRLF series, **Qin ShiCheng’s v4 repack fix** (now production-ready), and **Patrick Steinhardt’s unconditional approval** of the object storage extension series. Architectural debates flared over **`git reflog` usability** and **community voting eligibility**, while procedural clarifications dominated smaller threads.

---

## Notable threads

### 1. macOS CRLF conversion: RFC optimization consolidates `ie_match_stat()`
**What changed?**
Torsten Bögershausen proposed an RFC patch to Curtis Allen Smith’s `cs/read-cache-conversion-size` series, relocating the `would_convert_to_git()` check from `ie_modified()` to `ie_match_stat()`. This eliminates redundant attribute lookups and reduces the code footprint (8 insertions, 11 deletions).

### Why it matters

The optimization preserves correctness while addressing inefficiency in the v5 implementation, but **does not resolve the architectural debate** among five competing approaches (Curtis’s v5, Torsten’s RFC, Johannes Sixt’s `git add` fix, and Junio’s lstat() optimization concern). The patch is narrowly scoped to performance refinement and does not alter the fix’s core logic or test coverage.

### Key details

- **Files**: `read-cache.c`
- **Impact**: Reduces double `would_convert_to_git()` calls; affects `git diff-files`/`git diff-index` via `ie_match_stat()`.
- **Status**: RFC; no maintainer decision yet.

---

### 2. Repack data-loss fix: v4 series production-ready
**What changed?**
Qin ShiCheng posted v4 of the repack race-condition fix, now **unblocked** after `ps/odb-files-alternates` merged. The redesign introduces `--ignore-pack-keep-from` to explicitly tell `pack-objects` which packs it may delete, eliminating `--honor-pack-keep` entirely.

### Why it matters

The series fixes a **critical data-loss race** where concurrent pushes could delete packs whose objects were never copied elsewhere. The v4 approach is **simpler and safer**, with **Taylor Blau’s Reviewed-by** for the final patch. Benchmark data shows negligible overhead in realistic scenarios.

### Key details

- **Files**: `builtin/pack-objects.c`, `builtin/repack.c`, `packfile.c`, `t/t5329-pack-objects-cruft.sh`, `t/t7700-repack.sh`, `t/t7703-repack-geometric.sh`.
- **New plumbing**: `--ignore-pack-keep-from=<file>` (scalable alternative to `--keep-pack`).
- **Behavior**: Alternate ODB packs are now packed (not skipped), trading disk space for safety.
- **Status**: Ready for `next`; no new objections.

---

### 3. CI tooling: v3 series tightens tool installation
**What changed?**
Harald Nordgren posted v3 of the CI tooling series, which makes Perforce, Git LFS, and JGit installation **mandatory** on relevant platforms (ubuntu-*, macos-* for Perforce). The series drops redundant error-handling logic, relying on `set -e` in `ci/lib.sh`.

### Why it matters

The stricter approach (agreed in prior discussion) **fails fast** if tool installation fails, reducing log noise and silent test skips. The preparatory cleanup (removing the macOS `xattr` step) is now **unblocked** after GitLab CI confirmed the change is safe.

### Key details

- **Files**: `ci/install-dependencies.sh`
- **New behavior**: Build fails immediately if required tools are missing.
- **Status**: Ready for final review; **Junio’s "real positive review"** still needed.

---

### 4. Object storage extension: Series approved, architectural debate unresolved
**What changed?**
Patrick Steinhardt approved the v3 object storage extension series, calling it **"looking good"** with only minor nits. The series introduces `extensions.objectStorage` to enable pluggable ODB backends (e.g., cloud storage).

### Why it matters

This is a **major step toward Git libification**, but **SZEDER Gábor’s architectural objection** remains unresolved: whether user-configurable backends are justified when the `files` backend’s payload only relocates `.git/objects`. Patrick’s approval focuses on mechanical correctness, not the broader design question.

### Key details

- **Files**: `odb/`, `setup.c`, `repository.c`, `builtin/{clone,init,repo,rev-parse}.c`.
- **New symbols**: `remote.<name>.fetchRefs`, `enum object_storage_format`.
- **Status**: Technically approved; **Junio’s decision pending** on the architectural debate.

---

### 5. Community voting process: Hybrid eligibility model proposed
**What changed?**
Phillip Wood proposed a hybrid eligibility model for the community voting process: require a **minimum number of commits over all time** (to ensure a track record) **and at least one recent commit** (to ensure ongoing engagement). This avoids rigid time windows while filtering out inactive contributors.

### Why it matters

The proposal **resolves the tension** between inclusivity (lower thresholds) and practicality (higher thresholds), addressing concerns from **brian m. carlson** (drive-by contributions) and **Weijie Yuan** (contributor discomfort). It’s a **scriptable middle ground** that could unblock the stalled AI contribution policy vote.

### Key details

- **Proposed thresholds**: All-time commits (e.g., 26, per Tuomas Ahola’s average) + recent activity.
- **Status**: Under discussion; no consensus yet.

---

## In brief
- **`git reflog` usability**: Junio pushed back on Harald Nordgren’s detached-HEAD refactoring, calling it a **workflow preference** rather than a bug. The architectural debate remains unresolved.
- **`git imap-send`**: Harald Nordgren agreed to address Patrick Steinhardt’s defensive coding suggestions for the tunnel process lifecycle fix.
- **`git mergeconflicts(7)`**: Junio marked the series as **ready for `next`** after final editorial alignment with Julia Evans.
- **`git worktree list --branch`**: Phillip Wood suggested standardizing on short branch names via `--format='--branch=%(refname:lstrip=2)'` to simplify implementation.
- **Test modernization**: Kaartic Sivaraam’s series to make `clar`-based unit tests ignore `-l`/`--long-tests` was **approved by Patrick Steinhardt** and queued by Junio.
- **Git for Windows 2.56.0(4)**: Johannes Schindelin announced a **security-focused bugfix release** addressing corrupt code-signatures and a brotli vulnerability (CVE-2026-111497).

---