# Git mailing list daily digest for 2026/10/06

## The day in brief

The Git mailing list saw **116 emails** today, with key developments in several long-running threads. The most consequential were:

- **Git 3.0 planning** took center stage as the Git Contributor's Summit notes were shared, revealing consensus on a two-stage rollout (2.99 → 3.0) with SHA-256 support and Rust inclusion, but unresolved questions about LTS status and Rust mandates.
- **Trace2 hardening** was abandoned after Derrick Stolee signaled the protections provided by `banned-die.h` were "incomplete and thus less exciting," marking a pivot in the thread’s trajectory.
- **CI improvements** advanced with Harald Nordgren’s v6 of the leak sanitizer series, now including a compromise solution for broken annotation links.
- **Documentation efforts** saw progress with Julia Evans’ beginner-focused tutorial and the `gitmergeconflicts(7)` guide, though open questions remain about conflict style preservation and beginner workflows.

## Notable threads

### Git 3.0 planning (Git Contributor's Summit notes)
**What’s changing?**
The Git project is planning a two-stage rollout for Git 3.0 (2.99 → 3.0), with SHA-256 support and Rust inclusion as key features. The 3.0 release will differ only in flipped defaults for breaking changes, minimizing disruption.

### Why it matters:

- **SHA-256 adoption** is confirmed for GitHub (GA November 2026) and GitLab (already public), but JGit and Bitbucket lag behind. Interoperability between SHA-1 and SHA-256 repositories is complete on brian m. carlson’s personal branch but not yet upstreamed.
- **Rust support** drew no objections, but no decision was made on whether it will be mandatory or merely allowed in 3.0.
- **LTS status** for 2.99 was proposed (Gentoo expressed interest), but no consensus was reached.

### Open questions:

- How to synchronize with downstream distribution cycles?
- Will SHA-256 adoption surface latent bugs in Git itself?
- Funding for JGit and gitoxide SHA-256 support (Google declined JGit; gitoxide may secure funding).

### Next steps:

Follow-up discussions on each technical track (SHA-256 interoperability, Rust integration, LTS planning) are expected.

---

### Trace2 hardening abandoned
**What’s changing?**
Derrick Stolee announced he is setting aside the RFC series (`ds/trace2-tolerate-failed-timestamp`) that aimed to eliminate `die()`-prone helpers in the trace2 subsystem. The series introduced `banned-die.h` to enforce a no-`die()` policy but was deemed too complex for the benefits it provided.

### Why it matters:

- The trace2 subsystem must not crash Git, even under memory pressure or system call failures. The abandoned series sought to replace banned functions (e.g., `xsnprintf()`, `xstrdup()`, `ALLOC_ARRAY()`) with defensive fallbacks.
- Jeff King (Peff) had proposed a ground-up rewrite of trace2 as the only way to fully eliminate `die()`-prone dependencies, but this was not pursued due to complexity.

### Key takeaway:

The thread’s pivot reflects the challenges of hardening low-level subsystems without introducing excessive complexity. The core goal (preventing trace2 crashes) remains, but the path forward is now less certain.

---

### CI improvements: Leak sanitizer v6
**What’s changing?**
Harald Nordgren posted v6 of the CI leak sanitizer series, reviving the abandoned second patch with a compromise solution. The series now:
1. Stops leak-sanitizer scripts at the first failure (`--immediate`) and annotates leaks with the test script name and sanitizer output.
2. Embeds file and line context directly in annotation text, avoiding the earlier regression where annotation links were broken unless the test file was modified in the same pull request.

### Why it matters:

- The compromise resolves the show-stopper issue from v5 (broken annotation links) while preserving usability improvements.
- Phillip Wood accepted the compromise, and the series is now a candidate for `next`.

### Files touched:

- `ci/lib.sh` (enables `--immediate` for leak sanitizer jobs)
- `t/test-lib-github-workflow-markup.sh` (new functions for finding test case lines and formatting annotations)
- `t/test-lib.sh` (reorders annotation logic to ensure failures are reported before `--immediate` exits)

---

### Documentation: Beginner-focused tutorial
**What’s changing?**
Julia Evans posted a two-patch series replacing the outdated `gittutorial` with a beginner-focused guide tested on 112 users. The new tutorial covers:
- Local repository creation and commits
- Pushing to a remote host (GitHub/GitLab)
- SSH authentication and troubleshooting

### Why it matters:

- The existing tutorial is 18 years old and outdated, while user testing shows the new approach works despite rough edges.
- Open questions remain about repository creation (start with `git init` locally or clone from a forge?) and authentication (how to handle SSH/HTTPS setup generically?).

### Files touched:

- `Documentation/gittutorial.adoc` (replaced)

---

### `gitmergeconflicts(7)` guide refinements
**What’s changing?**
The `gitmergeconflicts(7)` guide saw further refinements, including:
- A new "WHAT IS A MERGE CONFLICT?" section, endorsed by D. Ben Knoble for its clarity.
- Clarification on `git commit` vs. `git merge --continue`, with the guide now steering users toward the latter as the more modern choice.
- Junio C Hamano’s detailed examples demonstrating how `diff3`’s common ancestor reveals the *intent* behind conflicting changes, transforming it from a stylistic preference into an essential tool for correct resolution.

### Open question:

Should `git checkout -m` remember and restore the original conflict style (e.g., `diff3`) as a fallback when the user does not explicitly request a different style?

---

## In brief

- **`git fetch --prune-tags` bugfix**: Orgad Shaneh gently pinged Junio C Hamano for a substantive response on the series aligning implementation with documentation. The thread remains stalled pending Junio’s input or clarification from Ævar Arnfjörð Bjarmason (original implementer).
- **`git history` signing**: Junio C Hamano confirmed the literal placeholder `...` in the completion test is correct, resolving the last blocking question. The series is now unblocked and ready to graduate to `next`.
- **`post-worktree` hook**: Phillip Wood raised design concerns about Domen Kožar’s v3 1/2 patch, questioning the worktree identifier’s usefulness, the four-argument interface, and the timing of the `remove` event. Domen has not yet responded.
- **Incremental connectivity check**: Kristofer Karlsson and Patrick Steinhardt resolved Patrick’s lingering question about trust boundary detection, confirming the incremental mode reuses the same mechanism as the full check (`git rev-list --not --all`).
- **`git stash create` extensions**: Kazumasa Shigeta and Phillip Wood narrowed the scope of the proposed changes, ruling out `--patch` and positional pathspecs while endorsing `--pathspec-from-file` and `-m`/`--message`.
- **Packfile corruption fix**: Junio C Hamano accepted the test’s platform dependency as a pragmatic trade-off, noting it either reproduces the bug or silently passes.
- **`git repo structure`**: Patrick Steinhardt reviewed Mark C. Chu-Carroll’s v2 patch, questioning the commit message’s lack of motivation and the restrictiveness of the `setup_revision_opt` struct.
- **`git(1)` man page rewrite**: D. Ben Knoble approved Julia Evans’ v2 revision, which incorporates feedback to use `git help push` as an alternative to `git push --help`.
- **Push negotiation bug**: Jens Röcker reported a bug where a pre-push hook repacking the local object database causes Git to resend common history that was already advertised to the receiver. No maintainer response yet.
- **`git status`/`git push` enhancements**: Harald Nordgren posted a six-patch series to better handle rebased push branches, introducing more granular tracking of divergence and adjusting advice messages.
- **Test modernization**: Muhammed Dilshad A updated `t/t0450-txt-doc-vs-help.sh` to use Git’s helper functions `test_path_is_file` and `test_path_is_missing` instead of raw shell test assertions. Patrick Steinhardt approved the patch.
- **Clock-skew warning**: Junio C Hamano raised fundamental objections to Devi Srinivas Vasamsetti’s patch, questioning its usefulness and noting the warning cannot address the root problem (clocks on other machines may already be wrong).