# Git mailing list daily digest for 2026/11/10

## The day in brief
The Git mailing list saw **141 emails** today, with key developments including:
- **`git repo info` path keys series** stalled on `path.hooks` necessity and `path.cdup` usability concerns.
- **Git 3.0 planning** clarified: Google’s SHA-256 talk deferred due to speaker emergency, but JGit’s pluggable backend remains production-ready.
- **`git worktree` safety improvements** advanced with performance regression identified in `%(worktreepath)` extension.
- **Security release** (Git 2.56.1) announced, addressing six CVEs including remote code execution via malicious submodule gitfiles.
- **Test modernization** for `git fast-import` tests gained maintainer praise, with 20-patch series reducing `t9300` from 4000 to 1200 lines.

## Notable threads

### 1. `[PATCH v8]` series adding path-related keys to `git repo info`
**What changed?**
Phillip Wood reiterated objections to `path.hooks` necessity and `path.cdup` usability, confirming the naming convention resolution (underscores `_`) is now settled.

### Why it matters

The series is mechanically complete but risks abandonment due to lack of reviewer engagement. The unresolved objections:
- `path.hooks`: Phillip argues it’s unnecessary and encourages `git hook run` instead.
- `path.cdup`: Usability concerns about script portability when run from repository root.

### Key details

- **Subsystem**: `git repo info` command
- **Files touched**: `builtin/repo.c`, `Documentation/git-repo.adoc`, tests
- **Status**: At risk of being dropped from `seen` unless objections are resolved

---

### 2. Git 3.0 planning and Gerrit User Summit updates
**What changed?**
Daniele Sassoli clarified that Google’s SHA-256 talk was removed from the Gerrit User Summit due to a speaker emergency, not a change in priorities. Luca Milanesio’s Cassandra-based ODB backend talk remains scheduled.

### Why it matters

The clarification removes ambiguity about Google’s commitment to SHA-256 interoperability. Key takeaways:
- **JGit’s pluggable backend** is production-ready (validated by Google’s internal deployments and GerritForge’s global refdb).
- **SHA-256 support** remains resource-constrained, not technically infeasible.
- **Summit attendance**: Junio, Luca Milanesio, and Christian Couder confirmed.

### Key details

- **Timeline**: Git 3.0 target remains April 2027, but stabilization may require additional 2.99.x releases.
- **Rust portability**: Compiles successfully on Cygwin (unofficial toolchain), but NonStop remains untested.
- **Security process**: GitLab increasing staffing for report triage; Toon Claes and Johannes Schindelin collaborating on next release.

---

### 3. `git worktree list --merged` / `--no-merged` improvements
**What changed?**
Phillip Wood identified a **critical performance regression** in patch 3/4: the new `%(worktreepath)` implementation uses O(N*M) complexity (iterating over all worktrees for every ref) instead of the old O(1) hashmap-based lookup.

### Why it matters

The series aims to improve worktree safety and filtering but risks introducing a performance bottleneck. Key developments:
- **Architectural alternative**: Phillip proposed adding a `%(currentworktree)` atom to `ref-filter.c` to simplify logic.
- **Performance fix**: Suggested refactoring `branch.c` to use a hashmap for O(1) lookups.

### Key details

- **Subsystem**: `ref-filter.c`, `branch.c`, `builtin/worktree.c`
- **Files touched**: 4 patches updating worktree safety and filtering
- **Status**: Ready for review with performance issue blocking integration

---

### 4. Security release: Git 2.56.1 and maintenance releases
**What changed?**
Junio C Hamano announced coordinated releases addressing six CVEs, including:
- **CVE-2026-111760**: Remote code execution via malicious submodule gitfiles.
- **CVE-2026-113408**: Path traversal in files ref backend.
- **CVE-2026-114467**: Stack corruption from multi-signature commits.

### Why it matters

The vulnerabilities span critical areas (submodules, ref backends, merge machinery) with potential for denial-of-service or arbitrary code execution. Key details:
- **Backported**: Fixes available for all supported maintenance tracks (2.56.1 down to maint-2.43).
- **Contributors**: Toon Claes (protocol v2 symref validation), Karthik Nayak (multi-signature handling), Jeff King (30 patches for const-correctness and HTTP header redaction).
- **Git for Windows**: Johannes Schindelin released v2.56.1 with identical fixes.

### Key details

- **Files touched**: xdiff, `remove_signature()`, files ref backend, protocol v2, `fast-import`, submodule gitfile parsing.
- **Behavior changes**: Stricter symref validation, hardened `fast-import` features, defense-in-depth hardening (e.g., rejecting over-long cache entries).

---

### 5. Test modernization: Splitting `t9300-fast-import.sh`
**What changed?**
Junio C Hamano praised Karthik Nayak’s 20-patch series refactoring `t9300-fast-import.sh`, calling it "long overdue." The series splits the 4000-line monolithic test into smaller, self-contained files.

### Why it matters

The refactoring improves maintainability by eliminating hidden dependencies between tests. Key achievements:
- **Reduction**: `t9300` shrinks from ~4000 to ~1200 lines.
- **New files**: `t9301-fast-import-notes.sh`, `t9304-fast-import-marks.sh`, etc.
- **Setup helpers**: Introduced to replace shared state with fresh repositories.

### Key details

- **Files touched**: 20 patches touching only the test suite (`t/` directory) and `meson.build`.
- **Status**: Under review; Junio requested a rebase on two in-flight topics (`cc/early-scan-options` and `tj/drop-no-parseopt`).

---

## In brief
- **`git mktree --batch` test coverage**: Roger Koranteng sent v2 addressing Laveesh Bansal’s feedback (verifying `stderr` error messages).
- **Detached HEAD documentation**: Junio and Julia Evans exchanged phrasing proposals, with Junio favoring technical precision ("`HEAD` records a reference") and Julia advocating user-centric language ("`HEAD` is where Git stores the name of your current branch").
- **HTTP retry improvements**: Ravi Mistry proposed two options for structuring the default `http.retryAfter` value (10 seconds) in v3.
- **ODB refactoring**: Patrick Steinhardt’s series isolating the loose-object backend received surface-level reviews from Kaartic Sivaraam, focusing on header hygiene and documentation preservation.
- **`the_repository` removal**: Junio and Patrick Steinhardt discussed architectural refinements for `unpack-trees.c`, deferring boundary vs. internal consistency checks to future work.
- **`checkout.orphanedCommitWarning`**: Junio questioned the design, asking whether the expensive reachability check could be optimized rather than made optional.
- **Pluralization fix**: aeron-gh sent a patch using `Q_()` for the "Split into %d hunks" message in `add-patch.c` to enable correct Polish pluralization.
- **Early option-scanning API**: Azuama Odinaka Maximus sent an RFC series introducing a reusable API for `parse-options`, targeting `rev-list`’s hardcoded option list.