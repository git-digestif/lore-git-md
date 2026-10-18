# Git mailing list daily digest for 2026/10/17

## The day in brief
The Git mailing list saw significant activity around **OpenSSL compatibility fixes** in `imap-send`, a **performance regression fix** for shallow fetches, and **design discussions** about branch cleanup commands. A **new bug report** revealed a crash in `git apply --check`, while **Outreachy applicants** continued onboarding. The **Git 3.0 planning thread** received production-scale validation of JGit’s pluggable backend, and **refactoring efforts** to remove `the_repository` advanced.

## Notable threads

### OpenSSL compatibility and correctness fixes in `imap-send`
**What changed?**
Beat Bolli posted v3 of a three-patch series addressing OpenSSL compatibility and correctness in `git imap-send`. The series replaces deprecated TLS initialization functions, rewrites hostname verification, and adds forward-compatibility macros for OpenSSL 4.0.

**Why it matters**
The patches modernize `imap-send`’s TLS stack, ensuring compatibility with future OpenSSL releases. However, the second patch introduces a **critical security regression**: removing the `SSL_get_peer_certificate()` check creates a trivial MITM vector by allowing certificate stripping. The series remains blocked until this issue is resolved.

**Key details**
- Patch 1/3: Mechanical API swap from `SSLv23_method()` to `TLS_method()` (uncontroversial).
- Patch 2/3: Replaces manual hostname verification with OpenSSL’s `SSL_set1_host()`, but removes the certificate check, introducing a security regression.
- Patch 3/3: Adds OpenSSL 4.0 compatibility macros for `SSL_set1_host()` → `SSL_set1_dnsname()` (premature, as the series is blocked on the security issue).

**What’s next?**
The author must address the security regression in patch 2/3 before the series can proceed. The OpenSSL 4.0 macros, while uncontroversial, should be deferred until the blocking issue is resolved.

---

### Performance regression fix for shallow fetches over protocol v2
**What changed?**
Harald Nordgren posted an RFC patch fixing a severe performance regression in `git fetch` from shallow repositories over protocol v2. Fetches could take minutes instead of seconds due to inefficient history traversal when advertising shallow commits.

**Why it matters**
The regression affects users of shallow repositories with long histories (e.g., 900k commits), making fetches impractically slow. The fix aligns protocol v2 behavior with protocol v0 by directly advertising shallow grafts when no specific depth is requested, avoiding redundant traversal.

**Key details**
- **Before**: Protocol v2 traversed the entire history to determine shallow commits, repeatedly visiting the same commits in merge-heavy histories.
- **After**: Protocol v2 advertises all shallow grafts directly, reducing fetch time from minutes to seconds.
- **Files touched**: `shallow.c` (depth comparison logic), `upload-pack.c` (graft advertisement).
- **Edge cases**: Preserves correctness for `git fetch --unshallow` by optimizing the traversal to visit each commit once.

**What’s next?**
The patch is marked as RFC, seeking feedback on correctness. Given the clear performance improvement (7 minutes → 1 second) and minimal diff (4 insertions, 4 deletions), it is likely to be accepted once reviewed.

---

### Design discussion: `git branch --delete-merged` and squash/rebased branches
**What changed?**
Harald Nordgren and Junio C Hamano continued a design discussion about extending `git branch --delete-merged` to detect squash-merged and rebase-merged branches. Junio proposed a **fundamental CLI redesign**: decouple the strict `--delete-merged` (tip-reachability only) from a new `--delete-irrelevant` (change-based detection for rebases, squash merges, and cherry-picks). Harald rejected this split, arguing that once a branch’s changes are upstream (regardless of how), the branch is no longer needed.

**Why it matters**
The discussion highlights a tension between **safety** (strict, conservative behavior) and **practicality** (deleting branches once their changes are upstream). The outcome will shape how users manage branches in workflows involving squash merges or rebases.

**Key details**
- **Harald’s position**: Treat squash and rebase detection as a unified feature under `--delete-merged`; once changes are upstream, the branch is irrelevant.
- **Junio’s position**: Split into `--delete-merged` (strict, tip-reachability only) and `--delete-irrelevant` (looser, change-based detection) to avoid ambiguity and preserve safety.
- **Algorithmic considerations**:
  - **Rebase detection**: Junio suggests reusing `git range-diff` logic; Harald agreed to explore incorporating authorship information as a performance filter.
  - **Squash detection**: Currently uses path-by-path three-way merge approximation, which lacks rename detection and is intentionally lenient.

**What’s next?**
The discussion remains at an impasse. Junio’s proposal introduces a new CLI contract, while Harald’s approach prioritizes workflow efficiency. A third-party perspective or maintainer decision may be needed to break the deadlock.

---

### Bug report: `git apply --check` crashes on new-file patches
**What changed?**
Gingrspacecadet reported a crash in `git apply --check` when processing a patch that creates a new file. The crash occurs with an assertion failure (`patch->is_new <= 0`) in `apply.c`’s `check_preimage` function.

**Why it matters**
The crash prevents users from checking new-file patches before applying them, breaking a common workflow. The assertion suggests a logic error in how the apply machinery handles the `is_new` flag for new-file patches.

**Key details**
- **Trigger**: `git apply --check` on a patch that creates a new file (e.g., `/dev/null` → `tests/constraint_type_signature_lifting.test`).
- **Assertion**: `patch->is_new <= 0` (code assumes a patch cannot be both "new" and have a preimage).
- **Subsystem**: `apply.c` (specifically `check_preimage`).
- **Reproducibility**: Deterministic; occurs regardless of the patch’s content.

**What’s next?**
The bug is clear and reproducible, warranting a fix. The correct approach may involve removing the assertion or adjusting the logic in `check_preimage` to handle new-file patches properly.

---

### Git 3.0 planning: JGit’s pluggable backend validated at scale
**What changed?**
Luca Milanesio provided production-scale validation of JGit’s pluggable backend architecture, addressing concerns raised during the Git Contributors’ Summit 2026. He cited two real-world examples:
1. **Google’s internal Git servers**, including `gerrit.googlesource.com`, which have used a proprietary JGit backend for over a decade.
2. **GerritForge’s open-source global refdb backend**, which powers multi-site Gerrit deployments with globally consistent ref updates.

**Why it matters**
The email shifts the narrative from "JGit is a gap" to **"JGit is production-ready but underutilized"**, removing technical barriers to Git 3.0’s object database (ODB) interoperability goals. The validation confirms that JGit’s pluggable backend is mature, scalable, and adaptable to Git 3.0’s ODB abstraction efforts.

**Key details**
- **Pluggable ODBs**: JGit’s infrastructure is production-ready, though Git’s implementation may still face unresolved tradeoffs (e.g., content-defined chunking).
- **SHA-256 interoperability**: Corporate interest exists (Google), but resource constraints remain the bottleneck.
- **Gerrit User Summit**: Scheduled for **November 10, 2026**, with Junio and Luca confirmed to attend. The event may provide further clarity on SHA-256 timelines.

**What’s next?**
The validation provides actionable context for Git 3.0’s ODB goals, but resource constraints (not technical barriers) remain the primary obstacle for SHA-256 interoperability. The Gerrit User Summit is the next milestone for coordination.

---

### Refactoring: Removing `the_repository`
**What changed?**
Two patches advanced the ongoing effort to remove the global `the_repository` variable:
1. Okhuomon Ajayi refactored `merge-blobs.c` to stop using `the_repository`.
2. Okhuomon Ajayi refactored `checkout.c`’s `unique_tracking_name()` to accept an explicit repository parameter.

**Why it matters**
The `the_repository` removal effort aims to make Git’s codebase more modular by explicitly passing repository context instead of relying on global state. This improves maintainability and reduces hidden dependencies.

**Key details**
- **Patch 1**: `merge-blobs.c` now passes the repository pointer explicitly through the call chain, removing the `USE_THE_REPOSITORY_VARIABLE` macro.
- **Patch 2**: `unique_tracking_name()` in `checkout.c` now accepts a `struct repository *` parameter, with callers passing `the_repository` for now.

**What’s next?**
The patches are mechanical and uncontroversial, following the established pattern of the `the_repository` removal effort. They are likely to be accepted once reviewed.

---

### Outreachy onboarding and microprojects
**What changed?**
Two Outreachy applicants introduced themselves and sought microprojects:
1. **Muritala Mudi** completed onboarding (building Git, running `t0040-parse-options.sh`) and is seeking a microproject in the parse-options subsystem.
2. **Keryle (Keryelefondji)** posted v3 of a patch migrating `git var` to the `parse-options` API.

**Why it matters**
Outreachy provides a pathway for new contributors to engage with the Git project. The parse-options subsystem is a common target for microprojects, offering a well-scoped introduction to Git’s codebase.

**Key details**
- **Mudi’s progress**: Built Git, ran parse-options tests, and followed the *MyFirstContribution* guide.
- **Keryle’s patch**: Migrates `git var` to `parse-options`, enforcing mutual exclusivity between `-l` and positional arguments. The v3 patch addresses metadata nits (author name, email).

**What’s next?**
Mudi will likely propose a microproject in the parse-options area, while Keryle’s patch is ready for review.

---

## In brief
- **`git repo structure` SHA-256 compatibility**: Junio identified a SHA-256 compatibility issue in the new test, which hardcodes 40-character SHA-1 object IDs. The author must address this before Git 3.0.
- **`git branch --delete-merged` for gone-upstream branches**: DonQuinleone proposed a new `--into <ref>` modifier to handle branches whose upstream has been deleted. The RFC suggests two shapes: a modifier to `--delete-merged` or composable filters like `--upstream-gone --merged=origin/master`.
- **`git format-rev` deprecation**: Kristoffer Haugsbakk clarified that `git diff-tree -z` uses separators, not terminators, making it unsuitable for incremental parsing in scripts. The functional gap between `git format-rev` and `git diff-tree` remains unresolved.
- **`git var` parse-options migration**: Keryle posted v3 of a patch migrating `git var` to the `parse-options` API, addressing metadata nits.
- **`git difftool` l10n fix**: Ammaar Bakshi fixed a missing `_()` wrapper for an error message in `git difftool --dir-diff`, enabling translation.
- **`git apply` crash**: Gingrspacecadet reported a crash in `git apply --check` when processing new-file patches, with an assertion failure in `apply.c`.
- **Index lock timeout**: Jun Wu posted v2 of a series introducing a configurable timeout (`core.indexLockTimeout`) for Git’s index lock, with fixes for pre-existing write races in `git commit` and `git reset`.
- **Integer-overflow hardening**: Junio rejected Simon Richter’s `off_t` suggestion for `http.c`’s `posn` variable, reinforcing `size_t` on memory-safety grounds.
- **Loose ref cache inconsistency**: Andy Koppe posted v2 of a patch fixing an inconsistency in how root refs are handled in the loose ref cache.
- **CI maintenance**: Junio updated GitHub Actions macOS runners from `macos-14` to `macos-15` to address a deprecation notice. Brian m. carlson endorsed the change.