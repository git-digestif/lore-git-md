# Git mailing list daily digest for 2026/10/19

## The day in brief
Junio C Hamano emphasized performance trade-offs in the CRLF conversion fix, Aleksei Sviridkin pinged a stalled `pack-objects` bugfix, and Luca Milanesio announced JGit talks at the upcoming Gerrit User Summit. The incremental connectivity check series cleared final review hurdles, while the `imap-send` OpenSSL series faced new security and design concerns. Documentation updates for `gittutorial` and partial-clone accuracy advanced, and a new `--follow` performance fix for merge commits was proposed.

## Notable threads

### CRLF conversion size-trusting bug: performance vs. correctness
Thread root

### What changed?

Junio C Hamano agreed that moving the conversion check to `ie_match_stat()` might be architecturally cleaner than the current `ie_modified()` placement, but warned that the patch must preserve Git’s lstat()-based optimization that avoids unnecessary content comparisons for unchanged files.

### Why it matters

The series aims to fix a macOS-specific bug where Git incorrectly reports file modifications after CRLF conversion alters file size. The architectural debate now centers on balancing correctness with the long-standing performance optimization that skips content comparisons for files whose sizes haven’t changed. Junio’s latest review frames the lstat() optimization as a baseline requirement, implying the current design may not yet meet the project’s standards for efficiency.

### Key details

- **Subsystem**: `read-cache.c` (index refresh logic)
- **Impact**: Affects all commands that refresh the index (e.g., `git status`, `git rebase`)
- **Performance data**: Curtis Allen Smith’s benchmarks show negligible overhead in realistic scenarios (12 microseconds per modified 2.6 KB file) but measurable costs in extreme cases (138ms for 10,000 modified files)
- **Open questions**: Whether to adopt Torsten Bögershausen’s proposal to move the check to `ie_match_stat()`, or pursue a hybrid approach that reconciles correctness with performance

---

### JGit readiness for Git 3.0: production validation and summit talks
Thread root

### What changed?

Luca Milanesio announced two talks at the upcoming Gerrit User Summit (November 9–10, 2026) that directly address Git 3.0’s ecosystem compatibility goals:
1. **Xing Huang (Google)**: "SHA-256 in JGit: Why, What, and When" (November 10, 2:00 PM PDT)
2. **Luca Milanesio**: "JGit Object Storage Backend with Apache Cassandra" (November 9, 1:30 PM PDT)

### Why it matters

The emails shift the narrative from "JGit is a gap" to **"JGit is production-ready and actively evolving"**, providing concrete evidence of its pluggable backend architecture’s maturity and scalability. Luca’s Cassandra prototype demonstrates JGit’s adaptability to modern distributed storage systems, while Google’s internal deployments (including `gerrit.googlesource.com`) validate the architecture’s long-term viability. The talks may clarify timelines and resource constraints for SHA-256 support in JGit, a key dependency for Git 3.0’s interoperability goals.

### Key details

- **Production validation**: Google’s internal Git servers have used a proprietary JGit backend built on the same open-source pluggable interfaces for over a decade
- **Resource constraints**: SHA-256 support in JGit is technically feasible but not actively developed due to prioritization; corporate interest (Google) exists but has not translated into action
- **Summit logistics**: Junio C Hamano, Luca Milanesio, and Christian Couder are confirmed to attend, providing a venue for direct coordination between Git and JGit maintainers
- **Git 3.0 timeline**: The provisional April 2027 target remains tentative, with Junio emphasizing that the stabilization period for Git 2.99 may require additional maintenance releases

---

### Incremental connectivity check: final review and performance trade-offs
Thread root

### What changed?

Patrick Steinhardt accepted the **1,000-commit cutoff** as a reasonable heuristic after seeing benchmark data across small (`git.git`), medium (`linux.git`), and large (proprietary) repositories. The data showed incremental mode is **2.6–2.8x faster** at 1,000 commits for medium/large repos, while small repos saw a **~30% slowdown** (but absolute times remained negligible). Patrick also endorsed the **three-mode config design** (`full`/`incremental`/`auto[=<n>]`) and agreed to centralize `die()` calls at the subprocess boundary to align with Git’s libification goals.

### Why it matters

The series introduces an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental` or `auto`) to reduce the cost of verifying object connectivity during fetches and pushes. The feature shifts cost from repository size to incoming change size, with a configurable cutoff to balance performance and safety. The final review clears the way for integration, pending mechanical adjustments (cutoff configurability, `die()` refactoring).

### Key details

- **Performance**: Up to 190x faster for small pushes to large repos (14 MB vs. 3.4 GB memory), 2.6–2.8x faster for large/medium repos at 1,000 commits
- **Config design**: `transfer.connectivityCheck=full|incremental|auto[=<n>]` (default: `full`), `transfer.connectivityCheckIncrementalMaxCommits=<n>` (default: 1,000)
- **Implementation**: New `tree-verify.c` (316 lines) implements inductive trust algorithm; parent trees trusted, new trees verified only where differing from parents
- **Test coverage**: `t5413-connectivity-check.sh` (680 lines) verifies semantic equivalence with full check, corruption detection, and promisor object handling
- **Open question**: Stack overflow risk in recursive tree verification (acknowledged as a potential follow-up)

---

### `imap-send` OpenSSL compatibility: security regression and design flaws
Thread root

### What changed?

Patrick Steinhardt raised three critical concerns blocking the OpenSSL 4.0 compatibility macros in Beat Bolli’s v3 series:
1. **Procedural issue**: The macros should have been introduced in the same commit as the `SSL_set1_host()` call (patch 2/3), not as a separate follow-up
2. **Correctness issue**: The patch assumes `cfg->host` is always a DNS name, but OpenSSL 4.0 splits hostname verification into `SSL_set1_dnsname()` (DNS names) and `SSL_set1_ipaddr()` (IP addresses). If `cfg->host` could ever be an IP address, the current patch would silently misbehave
3. **Clarity issue**: The error message conditionally changes the function name, which adds no value and could confuse debugging

### Why it matters

The v3 series is **blocked on four fronts**: the security regression (certificate stripping), the behavioral regression (redundant `verify_peer()` call), the process issues (lack of cover letter and range-diff), and now the OpenSSL 4.0 compatibility concerns. The security regression remains the primary technical blocker, but Patrick’s feedback adds a new layer of scrutiny to the patch’s design and maintainability.

### Key details

- **Security regression**: The removal of the `SSL_get_peer_certificate()` check creates a trivial MITM vector by allowing certificate stripping
- **Behavioral regression**: The redundant `verify_peer()` call after `SSL_connect()` fails under `SSL_VERIFY_PEER` is dead code
- **Test coverage**: No tests exist for this security-sensitive code path; Patrick highlighted the absence as a critical gap
- **OpenSSL 4.0**: The compatibility macros are now blocked by procedural, correctness, and clarity issues, and remain premature given the unresolved security regression

---

### `git log --follow` performance regression for merge commits
Thread root

### What changed?

Miklos Vajna posted a patch to fix a severe performance regression in `git log --follow` introduced by commit 304812ed33 ("log: improve --follow following renames for non-linear history"). The regression caused a test case on linux.git to balloon from ~28s to ~6m43s; the patch reduces this to ~37s while also fixing edge cases.

### Why it matters

The patch addresses a real-world performance pain point for users working with large repositories and merge-heavy histories. The fix eliminates the expensive `propagate_follow_pathspec_to_parent()` function, which diffed each commit against *all* of its parents with rename detection enabled, and instead reuses the diffs already being computed in `log_tree_diff()`’s parent loop.

### Key details

- **Performance**: Test case improves from 6m43s to 37s
- **Edge case handling**: Correctly follows renames on the side of a merge that wasn’t taken
- **Implementation**: New `follow_needs_renames()` helper computes merge bases only when necessary, avoiding expensive operations for back-merges where the path didn’t exist on one side
- **Test coverage**: New test cases verify the fix for merge commits with `-c`, `--cc`, or `--remerge-diff`

---

### Documentation updates: `gittutorial` and partial-clone accuracy
Thread roots

### What changed?

1. **`gittutorial` replacement**: Julia Evans posted v2 of a beginner-focused guide, incorporating feedback on platform assumptions, accessibility, and wording. The new tutorial is split into two parts (local repository creation and pushing to a remote host) and is based on user testing with 112 participants.
2. **Partial-clone accuracy**: Aleksei Sviridkin reposted a documentation patch with an improved commit message, clarifying that the "one-at-a-time fetch" limitation applies only to the object-lookup fallback in `odb.c` (1 of 11 call sites of `promisor_remote_get_direct()`).

### Why it matters

- The `gittutorial` update modernizes Git’s onboarding experience, addressing outdated references to CVS-like workflows and lack of forge integration. The patch is uncontroversial and likely to be merged soon.
- The partial-clone patch corrects a long-standing inaccuracy in the technical design document, improving the accuracy of Git’s documentation around dynamic object fetching. Junio previously stalled the patch due to an overly verbose cover letter, but the v2 iteration addresses this concern.

### Key details

- **`gittutorial`**:
  - **Files touched**: `Documentation/gittutorial.adoc` (replaced)
  - **Design choices**: Starts with `git init` (to avoid forge-specific confusion), uses SSH for authentication (with security disclaimers), omits branching/collaboration to keep scope manageable
  - **Open questions**: Whether to start with `git init` or `git clone`, and how to handle authentication generically without tying instructions to specific forges
- **Partial-clone**:
  - **Files touched**: `Documentation/technical/partial-clone.adoc`
  - **Behavior change**: Narrows the scope of the "one-at-a-time" limitation to the fallback path in `odb.c`; updates subprocess name from "fetch-pack" to "git fetch" to match implementation

---

## In brief
- **`pack-objects` bugfix**: Aleksei Sviridkin pinged the list about a stalled bugfix for inefficient fetching of missing objects in partial clones. Thread
- **ZIP archive date-handling**: René Scharfe’s patch to omit the Extended Timestamp (UT) extra field for out-of-range commit times is ready for integration after a test infrastructure fix. Thread
- **`git stash` shell completion**: D. Ben Knoble’s series to improve `git stash` completion faces design questions about Zsh-specific fixes and the removal of deprecated `stash save` completion. Thread
- **Index lock timeout**: Jun Wu’s configurable timeout for index locks (`core.indexLockTimeout`) sparked debate about configurability vs. root-cause fixes. Thread
- **Connectivity check for deepening fetches**: Igor Wiedler’s fix for a performance regression in shallow fetches over protocol v2 is ready for integration after addressing a Windows CI failure. Thread
- **Reftable backend buffer overflow**: Matthew Suozzo’s patch to fix a potential buffer overflow in the reftable backend’s object record decoder faces design questions about the allocation helper. Thread
- **Loose ref cache inconsistency**: Andy Koppe’s patch to fix an inconsistency in how root refs are handled in the loose ref cache is ready for integration, pending resolution of an architectural debate. Thread
- **ODB race condition**: Justin Tobler’s series to fix a race condition in `.keep` lockfile cleanup received final approval and is ready for integration. Thread
- **GitLab CI whitespace jobs**: Patrick Steinhardt’s patch to fix shallow clone failures in GitLab CI jobs sparked a debate about the jobs’ long-term value. Thread
- **GitHub Actions macOS runners**: Junio C Hamano posted v2 of a CI update to split macOS jobs between `macos-15` and `macos-26`, following Patrick Steinhardt’s suggestion. Thread
- **`upload-pack` performance regression**: Harald Nordgren’s patch to fix a performance regression in shallow fetches over protocol v2 needs revision to address a protocol-compliance issue. Thread
- **`git-rebase` documentation**: Omri Sarig posted v2 of a patch to correct outdated and missing option links in the `git-rebase` man page. Thread
- **`git bundle` memory safety**: Olof Fairbrook’s patch to fix a memory safety issue in `git bundle create` was approved and queued for integration. Thread
- **`the_repository` removal**: Shashank Srikumar’s patch to eliminate implicit use of `the_repository` in `prune_packed_objects()` received final clarification from Junio C Hamano. Thread
- **Rust integration**: Andy Koppe’s patch to fix a Makefile bug for custom Cargo target directories sparked a debate about implementation mechanism. Thread
- **Remote configuration**: Colin Hinton’s patch to defer validation of `remote.<name>.followRemoteHEAD` faces design questions about error message clarity and syntax unification. Thread
- **ZIP archive pluralization**: Keryle Fondji’s patch to use `Q_()` for plural-form error messages in the ZIP backend received feedback on commit message clarity. Thread
- **`git fsck` pluralization**: Aniket Gupta’s patch to use `Q_()` for the verbose "Checking connectivity" message was approved and queued for integration. Thread
- **`git status` triangular workflows**: Harald Nordgren’s patch to make `git status` compare against both `@{upstream}` and `@{push}` by default faces questions about test coverage and justification. Thread
- **Skipping negotiator correctness**: Taylor Blau posted a patch to fix a correctness issue in the skipping negotiator’s queue management. Thread
- **`MyFirstContribution` tutorial**: Junio C Hamano posted a patch to reorder the tutorial to emphasize review discussion before sending v2. Thread