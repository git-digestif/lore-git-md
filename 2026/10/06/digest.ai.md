# Git mailing list daily digest for 2026/10/06

## The day in brief
The Git mailing list saw **116 emails** today, with key developments including:
- **Maintainer buy-in** for a **merge conflict advice update** (Julia Evans), now ready for `next`.
- **Performance regression** in `test-lint` reported (Junio C Hamano), sparking discussion about trade-offs.
- **Push negotiation bug** with repacking hooks (Jens Röcker), confirmed via reproducible test case.
- **Git 3.0 planning** and **SHA-256 transition** debated at the Contributor Summit, with consensus on a two-stage rollout but unresolved questions about LTS status and Rust mandates.
- **Trace2 hardening series abandoned** (Derrick Stolee), citing complexity trade-offs.
- **CI leak sanitizer improvements** accepted (Harald Nordgren), resolving a critical regression.
- **`git stash create` extension** design refined (Kazumasa Shigeta, Phillip Wood), ruling out `--patch` and positional pathspecs.

---

## Notable threads

### 1. Merge conflict advice update (`git merge --continue`)
**Thread**: [PATCH] commit: warn on clock skew during commit creation
**Author**: Julia Evans
**Status**: Ready for `next` after Junio’s approval.

Julia Evans updated the merge conflict advice message to suggest `git merge --continue` instead of `git commit`, aligning it with advice for rebase, revert, and cherry-pick. The patch is uncontroversial and received **maintainer approval** from Junio C Hamano, who called it "better late than never." Phillip Wood suggested a minor grammatical tweak (*"to conclude the merge"* instead of *"to conclude merge"*), which Julia incorporated.

**Key details**:
- **Files touched**: `wt-status.c`, `t7060-wtstatus.sh`, `t7512-status-help.sh`.
- **Behavior change**: `git status` now advises `git merge --continue` during conflicts.
- **Impact**: Improves UI consistency across conflict-resolution workflows.

**Next steps**: Junio will queue the patch for `next`.

---

### 2. Performance regression in `test-lint`
**Thread**: Performance regression in `test-lint` (test-grep-lint)
**Author**: Junio C Hamano
**Status**: Under discussion; no fix proposed yet.

Junio reported a **5x slowdown** in the `test-lint` target, specifically the `test-grep-lint` check added in commit `c9a92e239f`. The check verifies `grep` invocations in the test suite use `--perl-regexp` (`-P`) where appropriate, but its implementation is suspected to be inefficient. The regression is noticeable during `make test` as a pause between cleanup and the first test output.

**Key details**:
- **Commit**: `c9a92e239f` (mm/test-grep-lint).
- **Subsystem**: Test suite linting (`t/Makefile`, `test-lint` target).
- **Impact**: ~50 s to run five times (up from ~9 s when reverted).

**Open questions**:
- Can `test-grep-lint` be optimized to restore the original performance?
- Is the trade-off between correctness (consistent `grep` behavior) and developer productivity acceptable?

**Next steps**: The thread is awaiting proposals for optimization or alternative implementations.

---

### 3. Push negotiation bug with repacking hooks
**Thread**: push negotiation bug: repacking hook causes redundant transfer of common history
**Author**: Jens Röcker
**Status**: Confirmed via reproducible test case; no fix yet.

Jens Röcker reported a bug where a **pre-push hook repacking the local object database** causes Git to resend common history that was already advertised to the receiver. The redundant transfer occurs regardless of `push.negotiate` setting, causing the receiver to store a new pack roughly the size of the historical blob (4 MiB in the test case). The issue affects both Apple Git 2.54.0 and upstream Git 2.56.0 on macOS/arm64.

**Key details**:
- **Subsystems**: Push negotiation, object database (ODB), pack generation, hooks.
- **Files implicated**: `send-pack.c`, `object.c`/`object.h` (`odb_has_object()` with `OBJECT_INFO_QUICK`).
- **Behavior**: Without the hook, transfer is minimal (≈300 B); with the hook, a 4 MiB historical blob is resent.
- **Test case**: Self-contained Python reproducer attached, verified via SHA-256 hash.

**Root cause hypothesis**: Stale pack catalogue in the parent process causes `odb_has_object(OBJECT_INFO_QUICK)` to miss repacked objects, leading the pack generator to walk history without excluding advertised bases.

**Next steps**: The thread is awaiting a proposed fix or further analysis of the mechanism.

---

### 4. Git 3.0 planning and SHA-256 transition
**Thread**: Git Contributor's Summit 2026 notes (Git 3.0 planning)
**Author**: Taylor Blau
**Status**: Consensus on two-stage rollout; unresolved questions about LTS and Rust.

The Git Contributor's Summit 2026 discussed the **Git 3.0 timeline**, with consensus on a **two-stage rollout**:
1. **2.99 release** (March 2027): Flips defaults for breaking changes but retains backward compatibility.
2. **3.0 release**: Immediately follows 2.99, differing only in flipped defaults.

**Key decisions**:
- **SHA-256 support**: Confirmed for GitHub (GA November 2026) and GitLab (already public); JGit and Bitbucket lag. Interoperability between SHA-1 and SHA-256 repositories is complete on brian m. carlson’s personal branch but not yet upstreamed.
- **Rust support**: No objections to inclusion in 3.0, but no decision on whether it will be mandatory or merely allowed.

**Open questions**:
- Should 2.99 be offered as an LTS (Gentoo expressed interest)?
- How to synchronize with downstream distribution cycles?
- Will SHA-256 adoption surface latent bugs in Git itself?
- Funding for JGit and gitoxide SHA-256 support (Google declined JGit; gitoxide may secure funding).

**Next steps**: Follow-up discussions on each technical track (SHA-256 interoperability, Rust integration, breaking changes).

---

### 5. Trace2 hardening series abandoned
**Thread**: [PATCH 0/7] trace2: tolerate failed timestamp formatting
**Author**: Derrick Stolee
**Status**: Abandoned due to complexity trade-offs.

Derrick Stolee **withdrew his seven-patch series** to harden the trace2 subsystem against `die()`-prone helpers (`xstrfmt`, `xcalloc`, `ALLOC_GROW`, `ALLOC_ARRAY`, `xstrdup`). The series introduced `banned-die.h` to enforce a no-`die()` policy in trace2 code but was deemed too complex and incomplete. Stolee cited the **unfavorable trade-off** between added complexity and the protections provided, noting that the header might give a false sense of security.

**Key details**:
- **Goal**: Prevent trace2 from crashing Git during telemetry operations (e.g., under memory pressure).
- **Withdrawn changes**: `banned-die.h`, defensive fallbacks for banned helpers.
- **Alternative**: No replacement proposed; the thread is now dormant.

**Next steps**: None; the series is abandoned.

---

### 6. CI leak sanitizer improvements accepted
**Thread**: [PATCH v6 0/2] ci: improve leak sanitizer failure reporting in GitHub Actions
**Author**: Harald Nordgren
**Status**: Accepted for `next` after Phillip Wood’s review.

Harald Nordgren’s **v6 series** improves leak sanitizer failure reporting in GitHub Actions by:
1. Stopping leak-sanitizer scripts at the first failure (`--immediate`) and annotating leaks with the test script name and sanitizer output.
2. Embedding file and line context directly in failure annotations as plain text, avoiding the earlier regression where annotation links were broken.

**Key details**:
- **Files touched**: `ci/lib.sh`, `t/test-lib-github-workflow-markup.sh`, `t/test-lib.sh`.
- **New symbols**: `find_test_case_line_`, `github_markup_script_name`.
- **Impact**: Improves CI debugging for Git contributors.

**Next steps**: Junio will queue the series for `next`.

---

### 7. `git stash create` extension design refined
**Thread**: [PATCH v3] repo: add revision-based filtering to 'git repo structure'
**Author**: Kazumasa Shigeta
**Status**: Design discussion; `--patch` and positional pathspecs ruled out.

Kazumasa Shigeta and Phillip Wood refined the design for extending `git stash create` with `--include-untracked` (`-u`) and `--all` (`-a`) options. The discussion ruled out:
- **`--patch`**: Not suitable for script-focused commands like `stash create`.
- **Positional pathspecs**: Unnecessary; `--pathspec-from-file` is sufficient for scripts.

**Key details**:
- **Proposed additions**: `--pathspec-from-file`, `-m`/`--message` for consistency with other Git commands.
- **Backward compatibility**: `PARSE_OPT_STOP_AT_NON_OPTION` preserves the positional-message grammar.
- **Next steps**: Awaiting a revised patch or further discussion on the subset of options to expose.

---

## In brief
- **`git fetch --prune-tags` bugfix**: Orgad Shaneh pinged Junio for a response on the stalled series.
- **`git history` signing series**: Junio confirmed the literal placeholder `...` in completion tests is correct.
- **`post-worktree` hook**: Phillip Wood raised design concerns about the unified hook’s interface and timing.
- **Incremental connectivity check**: Kristofer Karlsson and Patrick Steinhardt resolved trust boundary detection questions.
- **`gitbreaking-changes(7)` manpage**: Junio rejected a footnote-based compromise for message-ID formatting.
- **`gittutorial` removal**: Junio and Tuomas Ahola finalized procedural coordination for the side effect in `git(1)`.
- **`git(1)` intro rewrite**: D. Ben Knoble approved Julia Evans’s v2 patch.
- **`git repo structure` filtering**: Patrick Steinhardt reviewed Mark C. Chu-Carroll’s v2 patch, questioning the `setup_revision_opt` struct.
- **Packfile corruption fix**: Junio acknowledged the test’s platform dependency as a pragmatic trade-off.
- **`git stash` refactoring**: Junio identified a redundant index refresh in Phillip Wood’s patch.
- **`git repo` config key rename**: Karthik Nayak confirmed the backward-compatibility break is acceptable.
- **Test modernization**: Patrick Steinhardt approved Muhammed Dilshad A’s patch.
- **`git rev-parse` usage**: Alejandro Colomar asked for advice on resolving branch-or-commit arguments.
- **ZIP timestamp handling**: Matthew E. Luallen provided real-world tool behavior examples.
- **Clock-skew warning**: Junio questioned the feature’s usefulness, noting it cannot address the root problem.
- **Rebased push branches**: Harald Nordgren posted a six-patch series enhancing `git status` and `git push` for rebased branches.
- **Beginner tutorial**: Julia Evans posted a two-patch series replacing `gittutorial` with a beginner-focused guide.
- **SHA-256 interoperability**: Brian M. Carlson provided a workaround for converting repositories to SHA-256 and clarified the interoperability work’s status.