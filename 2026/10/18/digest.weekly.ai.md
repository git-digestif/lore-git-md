# The Git Project -- Weekly Digest for 2026/10/12 -- 2026/10/18

## The period in brief

The week of 2026/10/12--2026/10/18 saw **moderate to heavy traffic** (six active days) with a mix of **performance fixes, architectural improvements, and usability refinements**. The most consequential developments were the finalization of Tyler Cipriani’s `--force-if-includes` bugfix series, Harald Nordgren’s shallow-fetch performance fix, and Miklos Vajna’s proposed fix for a severe `git log --follow` regression. The `the_repository` removal effort continued to advance, while design debates around `git branch --delete-merged` and CRLF conversion checks revealed deeper architectural tensions.

**Do not miss**: the `--force-if-includes` bugfix series (now ready for merge), the shallow-fetch performance fix (RFC), and the ongoing `git log --follow` regression discussion.

---

## Key developments

### `--force-if-includes` bugfix series finalized
Tyler Cipriani posted **v7 of a three-patch series** fixing critical bugs in `git push --force-if-includes`. The series addresses two long-standing issues: (1) the feature incorrectly checks the reflog of the *local* branch whose name matches the *remote* destination (e.g., `refs/heads/main`), rather than the branch actually being pushed (e.g., `refs/heads/src` in `src:main`), and (2) it unnecessarily rejects fast-forward pushes of non-branch refs (tags, detached HEADs).

The v7 series is **technically complete**, with all prior feedback resolved. It introduces a new `ref->unverifiable` flag and actionable advice (`advice.pushRefUnverifiable`) for pushes lacking a reflog, ensuring consistent behavior across all transport layers. The series touches 12 files, including new test coverage for edge cases like differently named branches, HEAD-based pushes, and tags. Junio C Hamano has not yet queued the series, but it is ready for merge.

**Key participants**: Tyler Cipriani, Junio C Hamano, Jeff King.
**Status**: Ready for merge (v7).

---

### Shallow-fetch performance fix (protocol v2)
Harald Nordgren posted a **patch fixing a severe performance regression** in `git fetch` from shallow repositories over protocol v2. The issue causes fetches to take minutes instead of seconds due to inefficient history traversal when advertising shallow commits. The fix aligns protocol v2 behavior with protocol v0 by directly advertising shallow grafts when no specific depth is requested, avoiding redundant traversal. Harald reports the fix reduces fetch time from **7 minutes to 1 second** in a repository with 900k commits.

The patch is marked as **RFC**, seeking feedback on correctness. It touches `shallow.c` and `upload-pack.c` and is narrowly scoped to the shallow-fetch code path. The fix addresses a real-world performance problem affecting users of large, shallow repositories.

**Key participants**: Harald Nordgren, Junio C Hamano.
**Status**: Under review (RFC).

---

### `git log --follow` performance regression
Miklos Vajna proposed a **fix for a performance regression** in `git log --stat --follow` (bisected to commit 304813ed33), which slowed the command from 33 seconds to nearly 8 minutes in repositories with complex merge structures. The regression affects users tracing file history in large repositories (e.g., the Linux kernel). The fix optimizes `propagate_follow_pathspec_to_parent()` to skip redundant diffs while preserving rename-following behavior.

Junio C Hamano identified a **potential correctness issue**: the patch’s `has_same` flag may incorrectly skip rename detection in merge commits where a file is semantically renamed in one parent but syntactically unchanged in the merge. The discussion has shifted from performance to correctness, with no resolution yet.

**Key participants**: Miklos Vajna, Junio C Hamano, SZEDER Gábor.
**Status**: Under review; correctness concerns remain.

---

### `git branch --delete-merged`: squash merge detection debate
Harald Nordgren and Junio C Hamano debated the design of `git branch --delete-merged`, specifically whether the command should detect squash merges and rebases. Harald argues for a **unified `--delete-merged`** that treats branches as irrelevant once their changes are upstream, regardless of integration method. Junio proposes splitting the command into two options: a strict `--delete-merged` (tip-reachability only) and a looser `--delete-irrelevant` (change-based detection).

Harald’s experiment with `git range-diff` revealed it is **20× slower** than tree-based diffing, complicating the design. The debate remains unresolved, with no clear consensus on whether to prioritize strict correctness or workflow efficiency.

**Key participants**: Harald Nordgren, Junio C Hamano, D. Ben Knoble.
**Status**: Design discussion; no implementation yet.

---

### Configurable index lock timeout for IDE workflows
Jun Wu posted **v2 of a four-patch series** introducing a configurable timeout (`core.indexLockTimeout`) for Git’s index lock. The series addresses lock contention between background IDE `git status` calls and user-initiated commands (e.g., `git add`, `git commit`). The v2 series expands the scope to fix pre-existing write-race conditions in `git commit` and `git reset`, which could silently discard concurrent index updates.

The default timeout (1 second) remains open to debate, but the implementation is conservative, skipping the timeout for commands that read the index before locking. The series is well-structured and addresses prior feedback, making it a strong candidate for integration.

**Key participants**: Jun Wu, Jeff King, Junio C Hamano.
**Status**: Under review (v2).

---

### `git worktree repair` bugfix series (v3)
Yoichi NAKAYAMA posted **v3 of a five-patch series** fixing a bug in `git worktree repair` where the command could incorrectly modify `.git` files or gitdirs belonging to unrelated worktrees. The v3 update hardens the `get_worktree_id()` helper to handle non-canonical path separators (backslashes, duplicate slashes) in `.git` file contents, ensuring robust validation across platforms.

The series is narrowly scoped to `worktree.c` and the associated test script, with no changes to on-disk formats or user-visible behavior beyond the intended bugfix. The series is ready for broader review but has not yet been queued in `seen` or `next`.

**Key participants**: Yoichi NAKAYAMA, Junio C Hamano.
**Status**: Under review (v3).

---

### `git imap-send`: OpenSSL compatibility and correctness fixes
Beat Bolli posted **v3 of a three-patch series** addressing OpenSSL compatibility and correctness in `git imap-send`. The series replaces deprecated APIs (`SSLv23_method()` → `TLS_method()`) and rewrites manual hostname verification to use OpenSSL’s built-in `SSL_set1_host()` API. However, patch 2/3 introduces two critical regressions: a **security regression** (certificate stripping) and an **error reporting regression** (loss of hostname-specific error messages).

The v3 series includes a new patch (3/3) implementing a verification callback to restore lost error context, but it does not address the security regression. Patrick Steinhardt’s review identified both regressions and highlighted the lack of test coverage for this security-sensitive code path. The series is **not ready for merging** in its current form.

**Key participants**: Beat Bolli, Patrick Steinhardt, Xiangfei Ding.
**Status**: Blocked; security regression unresolved.

---

### CI infrastructure modernization lands in `next`
Patrick Steinhardt’s **eight-patch series modernizing Git’s CI configurations** was marked ready for `next`. The series removes outdated dependencies (Python 2, EOL Docker images), eliminates redundant jobs, and improves test coverage for reftable and SHA-256 combinations. The changes reduce CI complexity while maintaining or improving coverage, particularly for the SHA-256 transition.

The series also surfaced a latent issue where Rust had been accidentally disabled in CI for six months, which will be addressed in follow-up work.

**Key participants**: Patrick Steinhardt, Junio C Hamano.
**Status**: Queued in `next`.

---

### `git repo structure` revision filtering ready for `next`
Mark C. Chu-Carroll’s patch adding **revision-based filtering** to `git repo structure` was marked ready for `next`. The feature allows users to analyze specific subsets of the repository object graph, improving diagnostic capabilities for large repositories. The patch uses Git’s standard `setup_revisions()` machinery and adds a parameter to `count_references()` to avoid redundant ref-walks.

Junio C Hamano identified a **SHA-256 compatibility issue** in the new test, which hardcodes 40-character SHA-1 object IDs. The author will need to guard the test with a `SHA1` prerequisite or rewrite it to be hash-agnostic.

**Key participants**: Mark C. Chu-Carroll, Junio C Hamano, Patrick Steinhardt.
**Status**: Ready for `next`; test compatibility issue unresolved.

---

### `includeIf.hostname:<pattern>` implementation
Isabella Caselli posted **v2 of a three-patch series** implementing `includeIf.hostname:<pattern>` in Git’s configuration system. The feature allows users to conditionally include configuration files based on the machine’s hostname, addressing the use case of dotfile sharing across multiple machines. The implementation aligns with hosting platform conventions and provides a debugging aid via `git var GIT_HOSTNAME`.

Junio C Hamano described the implementation as "neat" and "quite nice," with no substantive objections. The only remaining question is whether hostname matching should be case-sensitive, case-insensitive, or support a `hostname/i:` variant.

**Key participants**: Isabella Caselli, Junio C Hamano.
**Status**: Under review (v2).

---

### Detached HEAD documentation rewrite
Julia Evans’s **rewrite of Git’s detached HEAD documentation** was merged. The new guide (`gitdetached-head(7)`) moves the explanation to a standalone man page, improving clarity and discoverability. The rewrite addresses four specific shortcomings in the current text: (1) it didn’t explain *why* detached HEAD is useful, (2) it took too long to define the concept, (3) it didn’t clarify how users might end up in this state accidentally, and (4) it included overly complex diagrams.

The patch updates advice messages to reference the new guide and touches `Documentation/git-checkout.adoc`, `advice.c`, and build system files.

**Key participants**: Julia Evans, Junio C Hamano, D. Ben Knoble.
**Status**: Merged to `master`.

---

### `gitmergeconflicts(7)` documentation series finalized
Julia Evans’s **`gitmergeconflicts(7)` documentation series** was finalized and merged. The new man page consolidates merge conflict resolution guidance, replacing duplicated advice across multiple command man pages. The guide includes:
- An exhaustive list of commands that can cause conflicts (`git merge`, `git rebase`, `git cherry-pick`, etc.).
- Side-by-side examples of `diff3` vs. default merge conflict format.
- Promotion of `merge.conflictstyle=diff3` as a best practice.
- Clarification of "ours" and "theirs" during rebases.

**Key participants**: Julia Evans, Junio C Hamano, Jeff King, Patrick Steinhardt.
**Status**: Merged to `master`.

---

### `git push` refspec matching optimization (10–100x speedup)
Jon Simons posted **v2 of a 12-patch series** replacing linear refspec traversals with `strmap` lookups, achieving **10–100x speedups** for large-scale branch deletions (e.g., pushing 100 delete refspecs to a server with 100k refs). The v2 series squashes failing-test commits into their corresponding fixes and adds four new patches for test coverage of server-side behaviors, refspec ordering guarantees, and performance baseline measurements.

The series addresses a real-world performance pain point in CI/CD pipelines and large monorepos. Benchmark timings show 100 delete refspecs dropping from **2.5–8.9s to 0.15–0.19s**.

**Key participants**: Jon Simons, Junio C Hamano.
**Status**: Under review (v2).

---

### `rerere` race-condition fix series ready for `next`
Thomas Bachem’s **two-patch series** introducing a configurable lock timeout (`rerere.lockTimeout`) and a `--skip-locked` mode for `git rerere gc` is now queued in `next`. The series fixes a long-standing race condition where concurrent `rerere` operations contend for the shared `MERGE_RR.lock`, exacerbated by the geometric maintenance strategy introduced in Git 2.54.

The series addresses all prior feedback, including Patrick Steinhardt’s suggestion to revert to a shared `RERERE_NOWAIT` flag. It improves the reliability of `rerere` operations, particularly in automated workflows.

**Key participants**: Thomas Bachem, Junio C Hamano, Patrick Steinhardt.
**Status**: Queued in `next`.

---

### `git blame` default ignore-revs feature approved for `next`
Ravi Mistry’s **feature series** making `git blame` automatically use `.git-blame-ignore-revs` (or `HEAD:.git-blame-ignore-revs` in bare repositories) when no explicit ignore file is configured has been approved for integration. The v6 revision addresses Junio’s polish-only feedback, replacing a "cute hack" with a clearer implementation using `.util = (void *)1` and an explanatory comment.

The series includes security hardening (NUL-byte rejection, promisor fetch prevention, and a 100MB size cap) and thorough test coverage. The feature aligns local behavior with hosting platforms like GitHub and GitLab.

**Key participants**: Ravi Mistry, Junio C Hamano.
**Status**: Approved for `next`.

---

### Incremental connectivity check for fetch/push
Kristofer Karlsson’s **four-patch series** introducing an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental`) saw significant review progress. The feature reduces the cost of verifying object connectivity during fetches and pushes by processing incoming commits in topological order and tracking trusted objects from parent commits.

Junio C Hamano signaled strong interest, and Patrick Steinhardt provided detailed feedback on implementation details, including type usage, error handling, and stack overflow risk. Benchmarks show up to **190x speedup** for small pushes to large repos.

**Key participants**: Kristofer Karlsson, Junio C Hamano, Patrick Steinhardt.
**Status**: Under active review.

---

### ODB alternates refactoring and optimization
Patrick Steinhardt posted **v2 of a six-patch series** refactoring and optimizing the alternates mechanism in the Git object database’s "files" backend. The series addresses performance inefficiencies in reading and streaming objects when alternates are present, then performs internal cleanup to simplify the ODB source API.

Justin Tobler provided substantive reviews, confirming the technical rationale and implementation for all five patches. He raised a design-level concern about the `ODB_FOR_EACH_OBJECT_LOCAL_ONLY` flag in the final patch, questioning whether all call sites have legitimate "files" backend-specific reasons.

**Key participants**: Patrick Steinhardt, Justin Tobler.
**Status**: Under review (v2).

---

### Reflog usability improvements
Harald Nordgren proposed a **three-patch series** adding `--operations` and `--historical-branch` options to `git reflog` and `git log -g` to improve usability. The current reflog shows every intermediate step of operations like rebase or branch switching, making it difficult to see the actual commands executed and which branch they affected.

The series:
1. Refactors hard-coded strings used to mark branch switches, renames, and rebase steps.
2. Implements `--operations` to collapse multi-step operations into single entries.
3. Implements `--historical-branch` to show which branch HEAD was on when each entry was recorded.

Kristoffer Haugsbakk raised questions about scope and design trade-offs, noting that `tig reflog` already mitigates some of the pain points the series aims to address.

**Key participants**: Harald Nordgren, Kristoffer Haugsbakk.
**Status**: Under review.

---

### Loose-ref iterator positioning fix
Venkat Ragavan posted a **bugfix** for a correctness issue in `cache_ref_iterator_seek()` where loose-ref iteration can misposition when a directory name shares a prefix with a ref name. The bug can cause `git` to return incorrect ref iteration results, potentially affecting any command that iterates over refs.

The patch fixes the logic in `refs/ref-cache.c` and adds regression tests covering loose, packed, and mixed ref storage. Junio C Hamano clarified the test structure, explaining the conditional setup is intentional and necessary for testing mixed storage modes.

**Key participants**: Venkat Ragavan, Junio C Hamano.
**Status**: Under review.

---

## In brief

**`git worktree repair` bugfix** -- Yoichi NAKAYAMA’s five-patch series fixing incorrect repair of unrelated worktrees saw substantive review from Junio, who probed the robustness of `get_worktree_id()` in handling non-canonical path separators. **Status**: Under review (v3).

**`git fetch --all --depth` regression** -- Rong Bao posted a patch fixing a regression in Git 2.56.0 where `git fetch --all --depth=<n>` or `git fetch --all --unshallow` fails to deepen the history when multiple remotes are configured. Junio raised concerns about whether the same propagation issue exists for submodule fetches. **Status**: Under review.

**`git pull --rebase` and custom fetch refspecs** -- Jyotish Kumar confirmed the issue is not yet being addressed by any in-flight topic and is still in the investigation phase. **Status**: Design discussion.

**`git stash` shell completion improvements** -- D. Ben Knoble’s five-patch series improving `git stash` shell completion, including a Zsh-specific bug fix for stash name completion (e.g., `stash@{0}`), saw design questions about preserving option autocompletion for the deprecated `save` subcommand. **Status**: Under review.

**`fast-import` crash fix** -- Marc Becker’s patch fixing a crash when resolving commits or tags created earlier in the same session received maintainer feedback on commit message clarity and test hygiene. **Status**: Queued for `master`.

**Windows credential helper parsing fix** -- Marc Becker’s v3 patch fixing multi-line secret parsing in the Windows credential helper (`contrib/credential/wincred`) addressed all prior feedback, including refactoring into helper functions and consistent wide-character handling. **Status**: Under review.

**ZIP archive timestamp handling** -- Matthew E. Luallen submitted a test infrastructure fix for René Scharfe’s ZIP archive patch, replacing `TIME_IS_64BIT` with `TIME_T_IS_64BIT` to ensure tests pass on 64-bit platforms and are skipped on 32-bit systems. **Status**: Under review.

**`imap-send` OpenSSL compatibility** -- Aniket Gupta identified a build-breaking deprecation warning in OpenSSL 4.1.0-beta2 due to the use of `ASN1_STRING_length` instead of `ASN1_STRING_get_length`. **Status**: Under review.

**`git mergeconflicts` documentation** -- Junio Hamano provided a squashable patch refining the `diff3` example in `Documentation/gitmergeconflicts.adoc` to clarify the pedagogical intent. **Status**: Merged.

**`git branch --delete-merged`** -- Harald Nordgren agreed to explore incorporating authorship information from `git range-diff` as a performance filter for rebase detection. **Status**: Design discussion.

**`git fetch` shallow tag backfilling** -- Junio Hamano raised three substantive questions about the patch’s logic, including whether the regression can occur when the main fetch transaction is empty. **Status**: Under review.

**Contribution guidelines** -- Junio Hamano queued a documentation patch updating `SubmittingPatches` to discourage splitting test and fix into separate commits. **Status**: Merged.

**Unicode character width tables** -- Beat Bolli posted a patch updating Git’s Unicode character width tables to Unicode 18, and Junio Hamano confirmed its correctness and queued it for integration. **Status**: Merged.

**`tig` pager configuration** -- Beat Bolli posted a patch adding syntax highlighting for the `Fixes:` and `Link:` trailers in the `tig` pager’s configuration file. **Status**: Under review.

**`git log --follow` regression** -- Miklos Vajna acknowledged the 14× slowdown introduced by commit 304813ed33, framing it as a trade-off for correctness. **Status**: Under investigation.

**`git checkout -m` conflict label preservation** -- Phillip Wood’s series to make `git checkout -m` recreate conflict markers using original branch names is queued in `next`. **Status**: Queued in `next`.

**Outreachy microprojects** -- Several applicants submitted patches for test modernization, i18n improvements, and completion updates, including pluralization fixes for `git describe` and `delta-islands`. **Status**: Under review.

**CI housekeeping** -- Patrick Steinhardt’s series modernizing Git’s CI configurations continues to receive feedback, with a naming dispute over the `linux-TEST-vars` job resolved by renaming it to `linux-feature-testbed`. **Status**: Queued in `next`.

**`git repo structure` revision filtering** -- Mark C. Chu-Carroll’s series adding revision-based filtering to `git repo structure` received feedback on revision-parsing defaults and test coverage. **Status**: Ready for `next` (SHA-256 compatibility issue unresolved).

**`git detached-head` documentation** -- Julia Evans’s rewrite of Git’s explanation of detached HEAD state is ready for integration, with the new guide named `gitdetached-head(7)` for consistency. **Status**: Merged.

**`git repack` dry-run regression** -- Patrick Steinhardt noted a potential code hygiene opportunity in Siddharth Shrimali’s fix for `git repack --drop-filtered --dry-run`. **Status**: Under review.

**`git log --graph` rendering bug** -- herr.kaste reported a rendering error in `git log --graph --all --oneline` where an orphaned commit is drawn with a spurious parent connection. **Status**: Under investigation.

**`git log` completion update** -- Mirko Faina added shell completion support for `--graph-indent` and `--no-graph-indent` to `git log`. **Status**: Under review.

**Index lock timeout feature** -- Jun Wu’s series adding a configurable timeout for Git’s index lock received feedback on race-avoidance logic and the user-hostility of the new config knob. **Status**: Under review (v2).

**Coverity-inspired integer-overflow fixes** -- Johannes Schindelin posted a five-patch series hardening Git against integer overflows. Phillip Wood raised concerns about the portability of the new `mult_overflows` macro. **Status**: Under review.

**`git help user-manual` bug** -- John Wilson reported a bug on Git for Windows where `git help user-manual` attempts to open `gituser-manual.html` instead of the correct filename `user-manual.html`. Junio confirmed the issue is upstream. **Status**: Under investigation.

**`git describe` l10n update** -- Raj Ojha posted a patch updating two debug messages in `builtin/describe.c` to use the `Q_()` pluralization macro. **Status**: Under review.

**`the_repository` removal** -- Okhuomon Ajayi posted refactoring patches advancing the ongoing `the_repository` removal effort by updating functions in `merge.c`, `checkout.c`, and `prune_packed_objects()` to use explicitly passed `struct repository *` parameters. **Status**: Under review.

**Cache-tree crash fix** -- Szymon Wilczek posted a bugfix preventing `git rebase` from crashing when another process holds the index lock during an in-process commit attempt. **Status**: Under review.

**`git format-rev` deprecation proposal** -- Kristoffer Haugsbakk proposed deprecating the experimental `git format-rev` command, arguing its core functionality is already provided by `git diff-tree --stdin --format=medium`. The discussion revealed a functional gap: `git diff-tree` does not support NUL termination for the `--format=medium` pretty-printing mode. **Status**: Under discussion.

**`git var` refactoring** -- Junio C Hamano requested a v2 to address cosmetic issues (sign-off placement, indentation). **Status**: Under review.

**`git apply --check` crash** -- Gingrspacecadet reported a crash in `git apply --check` with assertion `patch->is_new <= 0` when processing a new-file patch. **Status**: Under investigation.

**`git bundle create` bugfix** -- Olof Fairbrook posted a patch to avoid freeing uninitialized stack memory when given an unrecognized argument. **Status**: Under review.

**Config subsystem bugfix** -- Olof Fairbrook posted a patch to correctly handle high-bit bytes (0x80–0xff) in in-memory config blobs, fixing a long-standing issue where 0xff was misinterpreted as EOF. **Status**: Under review.

**Makefile bugfix** -- Andy Koppe posted a patch fixing a build failure when the user has a custom Cargo target directory. **Status**: Under review.

**`remote.<name>.followRemoteHEAD` validation** -- Colin Hinton posted a patch deferring validation until the setting is actually needed (during `git fetch`), eliminating spurious warnings. **Status**: Under review.

---

## Looking ahead

The next week is likely to see **continued review and integration** of the `--force-if-includes` bugfix series, the shallow-fetch performance fix, and the `git log --follow` regression fix. The **`git branch --delete-merged`** design debate may produce a prototype implementation, while the **CRLF conversion check** thread could see a revised patch addressing the architectural concerns raised by Johannes Sixt and Junio C Hamano.

**Ongoing efforts** like the `the_repository` removal, ODB abstraction, and reftable backend will continue to generate patches, with the latter potentially seeing progress toward production readiness. The **Rustification effort** remains a wildcard, with no major patches posted this week but ongoing discussion about platform support and memory safety.

**Outreachy contributions** will likely accelerate, with applicants focusing on test modernization, parse-options refactoring, and global state reduction. The **parse-options subsystem** is a particularly active target for microprojects, offering well-scoped opportunities for newcomers.