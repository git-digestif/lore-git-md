# Git mailing list daily digest for 2026/11/06

## The day in brief
The Git mailing list saw significant activity around **global config handling**, **integration pipeline policy**, and **bugfixes for critical subsystems**. Delilah Ashley Wu's global config series was accepted with minor tweaks, while Junio C Hamano formalized a new `seen` branch ejection policy to balance reviewer productivity with quality control. Notable bugfixes included fixes for `git apply` crashes, `git merge` ODB cache invalidation, and `git unpack-objects` type mismatches on 32-bit platforms. Outreachy proposals and CI optimizations also featured prominently.

---

## Notable threads

### Global config handling inconsistency fixed
**Thread root**: 2025/10/10/01-14-05
**What changed**: Delilah Ashley Wu's two-patch v3 series to fix Git's global config file handling inconsistency was accepted. The series ensures `git config list --global` reads both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`, aligning with unscoped operations.
**Why it matters**: This fixes a long-standing inconsistency where `--global` only read `$HOME/.gitconfig` while unscoped operations read both files. The implementation uses ignore flags and `do_git_config_sequence()` to restrict config reading to the global scope.
**Today's developments**:
- Junio C Hamano accepted the first patch ("t1300: test list with missing global config") with a minor question about test portability regarding XDG paths.
- The maintainer accepted the second patch ("config: read global scope via config_sequence") with a request for a defensive `BUG()` check in `git_config_system()` to ensure no caller passes a NULL `opts` pointer.

---

### Integration pipeline policy update
**Thread root**: 2026/07/27/03-09-12
**What changed**: Junio C Hamano adopted Phillip Wood's suggestion to explicitly frame ejection from the `seen` branch as a reversible, routine event rather than a final rejection.
**Why it matters**: This change addresses contributor anxiety by signaling that ejection is temporary and contingent on renewed engagement. The new wording in "What's cooking" reports emphasizes that topics can be resubmitted if they gain enough interest.
**Today's developments**:
- The maintainer updated the "Discarded" section wording to: *"Many of these topics were picked up prematurely but did not prove ready for inclusion, and will be removed. They can be resubmit if they gained enough interest and showed good fit to the system in the future, unless noted otherwise."*

---

### Submodule crash fix under review
**Thread root**: 2026/08/21/23-01-40
**What changed**: Volodymyr Vriukalo's two-patch bugfix series for a crash in `git branch --recurse-submodules` with non-ref start points received substantive review.
**Why it matters**: The bug manifests in repositories without submodules (BUG abort) and with submodules (truncated argument list). The fix prevents tracking a start point that has no ref and allows recursion to proceed when there is no tracking name.
**Today's developments**:
- D. Ben Knoble suggested simplifying the test case in the first patch by directly creating a branch at a non-ref start point (e.g., `git branch t @~3`) rather than first creating it at `HEAD~1` and then moving it.
- The reviewer confirmed the technical soundness of the second patch but recommended improvements to the commit message and in-code comments.

---

### Bundle-uri URI handling regression fixed
**Thread root**: 2026/10/31/15-57-00
**What changed**: Ravi Mistry's HTTP retry series resolved its primary blocker by agreeing to default `http.retryAfter` to **10 seconds** (with exponential backoff: 10s, 20s, 40s, ...).
**Why it matters**: This compromise balances server protection (brian m. carlson's priority) with interactive usability (Ravi's concern). The series adds exponential backoff for HTTP retries and extends retries to POST operations.
**Today's developments**:
- Ravi accepted brian's argument that a 1-second initial delay is too short for Git's computationally expensive requests and agreed to the 10-second default.

---

### Credential-cache daemon race condition fix
**Thread root**: 2026/11/01/23-29-24
**What changed**: Farid Zakaria's fix for a race condition in the Git credential-cache daemon acknowledged Junio's architectural concern about mixing `unix_ss_*` and `tempfile` APIs.
**Why it matters**: The race causes concurrent `git credential-cache store` commands to unlink each other's sockets, leading to lost credentials and orphaned daemons. The fix uses `unix_ss_create()` to safely handle concurrent socket creation.
**Today's developments**:
- Farid explained that he was unsure whether the complexity of handling the "stolen" socket case was justified, given that the race might only occur during Git upgrades (old client + new daemon).

---

### `git apply` crash regression fixed
**Thread root**: 2026/10/24/05-24-49
**What changed**: Hannes Brandstätter-Müller's fix for a crash in `git apply` when `core.fileMode=false` received substantive review.
**Why it matters**: The regression causes patch series containing a pure rename followed by an edit to fail with `error: renamed: wrong type`. The fix prevents tracking a start point that has no ref.
**Today's developments**:
- Aniket Gupta identified a potential security edge case: a malicious patch could change a regular file into a symlink when `core.fileMode=false` is set, and proposed a refined fix that masks `patch->old_mode` to preserve only the permission bits (`07777`) and explicitly sets the file type to regular (`S_IFREG`).
- Junio questioned whether the scenario (a pure rename followed by a type-changing patch) represents a *valid* patch input or merely corrupt data.

---

### Loose ODB cache invalidation bug fixed
**Thread root**: 2026/11/04/12-16-33
**What changed**: Patrick Steinhardt's fix for a bug where newly written merged blobs in `git merge` cannot be read back due to stale loose object cache entries was endorsed for `next`.
**Why it matters**: The bug causes Git to incorrectly assume objects are missing locally and trigger unnecessary, failing backfill fetches from promisor remotes. The fix ensures the loose object cache is properly invalidated after writing merged blobs.
**Today's developments**:
- Junio endorsed landing the current patch as a targeted regression fix while documenting the broader `QUICK` flag limitations via a `NEEDSWORK` comment in `odb.h`.

---

### `git unpack-objects` type mismatch fixed
**Thread root**: 2026/11/06/12-51-15
**What changed**: Kristofer Karlsson's fix for a type mismatch in `unpack-objects` that causes blob sizes larger than 4GB to be truncated on 32-bit platforms (notably Windows).
**Why it matters**: The regression was introduced in `f2063855fb` (May 2026) when object sizes were migrated to `size_t` but `stream_blob()` was left using `unsigned long`. The fix changes `stream_blob()` to accept `size_t`.
**Today's developments**:
- The patch was posted as part of a two-patch series, with the second patch adding test coverage.

---

## In brief
- **`git rerere` subcommand modernization**: Patrick Steinhardt's 12-patch series converting `git rerere` to use `OPT_SUBCOMMAND()` received surface-level review. Karthik Nayak suggested minor improvements to enum documentation and code structure.
- **`git log --follow` performance**: Patrick Steinhardt identified a potential regression in Miklos Vajna's performance series, where `follow_needs_renames()` disables rename detection for commits with unrelated histories, contradicting the intent of commit 304812ed33.
- **`git rev-list` option parsing**: Junio challenged the architectural premise of Azuama Odinaka Maximus's fix, suggesting eliminating the "early scan" entirely and solving the underlying problem by recording explicit option settings during normal parsing.
- **`git ls-files --type` proposal**: Alejandro Colomar's feature request for a `--type` filter option was rejected by Junio, who argued that `git ls-files` is a plumbing command and should not grow arbitrary filtering options.
- **Outreachy proposals**: Shashank S proposed implementing promisor remote fetch ordering, while Srujith Poojary targeted global state reduction in pathspec skip-worktree matching logic.
- **GitLab CI speedups**: Patrick Steinhardt's seven-patch series targeting GitLab CI bottlenecks received maintainer acknowledgment. Junio found all patches "interesting" and "fully made sense."
- **`git column` argument parsing**: Fien Dora proposed refactoring `git column` to use `early_scan_options()`, seeking maintainer input on whether to preserve current restrictions or expand to full `parse_options()` flexibility.
- **Git v3 compatibility**: Adam Dinwoodie reported that `git init` incorrectly defaults to SHA-1 even when Git is built with `WITH_BREAKING_CHANGES=1`. brian m. carlson confirmed the bug and is preparing a fix.