# Git mailing list daily digest for 2026/09/11

## The day in brief
The Git mailing list saw active discussion on several fronts today. Key developments include a **design debate** over the `receive-report` hook’s commit message verbosity, **substantive reviews** of ODB refactoring series (fsck pluggability, alternates handling), **CI/build system updates** (Debian 12 adoption, Rust toolchain configuration), and **bugfix patches** for `git blame`, `git range-diff`, and merge driver tempfile cleanup. The maintainer’s "What’s cooking" report highlighted progress on ODB abstraction and Rust integration, while a new feature proposal for `git blame` gained traction.

---

## Notable threads

### 1. `git history` signing series nears completion
**What changed?**
Patrick Steinhardt provided **surface-level review** on Souma’s v2 series teaching `git history` to sign rewritten commits. Feedback focused on commit message clarity and style nits rather than technical substance.

**Why it matters**
The series adds GPG signing support to `git history`’s `drop`, `fixup`, `reword`, and `split` operations, aligning them with `git commit`’s existing signing options (`-S`, `--gpg-sign`). This closes a gap in the interactive rebase workflow, where users previously couldn’t sign rewritten commits without manual intervention.

**Key details**
- **Files touched**: `builtin/history.c`, `replay.c`, `replay.h`, documentation, and tests.
- **New behavior**: Respects `commit.gpgsign`, `-S[=<key-id>]`, `--gpg-sign[=<key-id>]`, and `--no-gpg-sign`.
- **Open question**: Whether signing rewritten commits (originally authored by others) is conceptually sound. The commit message now discusses this trade-off.

**Status**: Ready for final review; minor cleanups pending.

---

### 2. `git repo info` path keys series faces edge-case challenges
**What changed?**
Junio C Hamano identified **two high-weight issues** in K Jayatheerth’s v6 series adding path-related keys to `git repo info`:
1. **`path.cdup` edge-case discrepancy**: When outside the working tree, `path.cdup` returns an empty string while `git rev-parse --show-cdup` returns the absolute path to the working tree root.
2. **Patch 2/7 refactoring requirement**: The second patch (introducing `path.superproject-root`) must be split into two: one to fix `get_superproject_working_tree()` and another to add the new keys.

**Why it matters**
The series exposes repository paths (e.g., hooks directory, index file) in a scriptable format, but the `path.cdup` discrepancy undermines its goal of consolidating path information. The refactoring request improves bisectability and clarity.

**Key details**
- **Files touched**: `builtin/repo.c`, `submodule.c`, `builtin/rev-parse.c`, documentation, and tests.
- **New keys**: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, `path.cdup`.
- **Fix required**: Modify `get_path_cdup()` to fall back to `get_git_work_tree()` when `repo->prefix` is empty.

**Status**: Awaiting v7 to address the edge cases.

---

### 3. ODB fsck pluggability series resolves architectural question
**What changed?**
Patrick Steinhardt accepted Toon Claes’s suggestion to move the `ODB_FSCK_FULL` filtering logic from the central `odb_fsck()` helper into the "files" backend’s `fsck` callback. This makes the infrastructure more flexible for non-local backends (e.g., reftable, cloud storage).

**Why it matters**
The series refactors Git’s fsck checks to be pluggable per ODB backend, preparing for future storage systems. The architectural change ensures the design can accommodate backends with different semantics for "full" verification.

**Key details**
- **Files touched**: `builtin/fsck.c`, `odb/source-files.c`, `odb/source-packed.c`, and tests.
- **New symbols**: `odb_source_fsck_fn`, `struct odb_fsck_options`, `enum odb_fsck_flags`.
- **Behavior**: No user-visible changes; backend-specific checks now respect `ODB_FSCK_FULL` independently.

**Status**: v3 posted; ready for final review.

---

### 4. CI/build system updates: Debian 12 and Rust toolchain
**What changed?**
- **Debian 12 adoption**: Junio C Hamano confirmed the patch to update CI from Debian 11 to Debian 12 is merged. The `linux32` job remains on Ubuntu 20.04 due to i386 support limitations.
- **Rust toolchain configuration**: Junio and James Le Cuirot clarified that `CARGO_BUILD_TARGET` should not be set for native builds, as it changes Cargo’s internal behavior. The series will be revised to drop the `--target` flag for native builds.

**Why it matters**
- **Debian 12**: Ensures CI runs on a supported distribution (LTS until 2028).
- **Rust toolchain**: Unblocks Rust compilation in GitHub Actions Windows CI, a prerequisite for the broader Rustification effort.

**Key details**
- **Files touched**: `.github/workflows/main.yml`, `.gitlab-ci.yml`, `ci/lib.sh`, `config.mak.uname`, `Makefile`.
- **New behavior**: Rust builds now target the GCC ABI (used by Git for Windows) instead of the default MSVC ABI.

**Status**: Debian 12 patch merged; Rust series awaiting v3.

---

### 5. `receive-report` hook series: Commit message verbosity debated
**What changed?**
Oswald Buddenhagen critiqued the commit message in patch 4/4 for duplicating documentation and obscuring the core rationale ("why?"). Karthik Nayak acknowledged the feedback but opted not to revise the message at this stage, citing the series’ near-completion.

**Why it matters**
The `receive-report` hook enables server-side filtering of push status reports, a key feature for GitLab’s MVCC system. The debate highlights a tension between thorough documentation and concise commit messages.

**Key details**
- **Files touched**: `builtin/receive-pack.c`, documentation, and tests.
- **New hook**: `receive-report` acts as a bidirectional filter for pkt-line encoded status reports.
- **Design trade-off**: The hook intentionally decouples reported status from actual repository state, violating Git’s traditional consistency guarantees.

**Status**: Ready for merge; commit message verbosity deferred as a cosmetic issue.

---

### 6. `git blame` feature proposal: Default ignore file
**What changed?**
Ravi Mistry proposed making `git blame` automatically look for a `.git-blame-ignore-revs` file in the repository root if no explicit ignore file is configured. This aligns local behavior with web interfaces (GitHub, GitLab).

**Why it matters**
The `.git-blame-ignore-revs` file has become an ecosystem standard, but local `git blame` invocations require manual configuration, leading to user confusion.

**Key details**
- **Files touched**: `blame.c`, documentation, and tests.
- **New behavior**: Automatically uses `.git-blame-ignore-revs` if no ignore file is configured.
- **Edge cases**: Handles symlinks, bare repositories, and CLI/config overrides.

**Status**: New proposal; awaiting review.

---

## In brief
- **`git range-diff --matched-only`**: Harald Nordgren’s feature patch was finalized with Junio’s documentation wording. The new option filters output to show only commits present in both input ranges.
- **Merge driver tempfile cleanup**: Jeff King’s series to fix SIGINT/SIGQUIT cleanup in external merge drivers received substantive review from Elijah Newren, who identified a latent type-safety bug. A v3 is expected.
- **`--force-if-includes` bugfix**: Tyler Cipriani addressed review feedback on the series fixing reflog checks for `git push --force-if-includes`. A v4 will generalize `HEAD` resolution and document deletion handling.
- **ODB alternates refactoring**: Patrick Steinhardt’s series to remove ad-hoc source linking was fast-tracked to `next` after resolving a memory leak and edge-case concerns.
- **Coccinelle rules**: Junio C Hamano removed a risky rule that converted `if (!E) free(E);` into `free(E)` and added a rule to allow unconditional `FREE_AND_NULL(E)` calls.