# Git mailing list daily digest for 2026/10/22

## The day in brief
The Git mailing list saw significant activity around **governance proposals** (voting process for contentious decisions), **performance optimizations** (fetch negotiation, `git log --follow`, index locking), and **bugfixes** (SSH signing verification, shallow fetches, worktree improvements). Key discussions included the **AI contribution policy stalemate**, which prompted a formal voting process proposal, and **index lock contention** in large repositories, with competing approaches to reduce IDE workflow disruptions. Several **Outreachy contributions** progressed, and a **terminology cleanup** series aligned documentation with 2024's "pseudoref" vs. "root ref" redefinition.

---

## Notable threads

### Governance: Voting process for contentious decisions
**What's changing?**
Emily Shaffer proposed a formal voting process to resolve contentious decisions (e.g., AI contribution policy) when consensus cannot be reached through discussion. The patch adds `Documentation/CommunityVoting.adoc` and updates `Documentation/DecisionMaking.adoc`.

**Why it matters**
The proposal directly addresses the **AI contribution policy stalemate**, where competing legal and cultural concerns have prevented consensus. It also establishes a template for future governance decisions, potentially reducing polarization in contentious debates.

**Today's developments**
- **Eligibility criteria refined**: Phillip Wood proposed counting *patch series* (not individual patches) and requiring substantive reviews (not just "LGTM") to qualify for voting. He also suggested limiting eligibility to contributors active in the past two years to prevent "dogpiling" by outsiders.
- **Secret ballots proposed**: Patrick Steinhardt and brian m. carlson advocated for secret ballots to mitigate concerns about corporate influence, shifting the debate from *whether* to vote to *how* to design voting fairly.
- **Corporate influence debated**: Kristoffer Haugsbakk argued that formal voting would inherently favor corporate interests (e.g., AI adoption), while brian m. carlson countered that contributors can vote independently of their employers.
- **Justin Tobler cautioned** that stricter eligibility criteria could disproportionately favor corporate-affiliated contributors, proposing a time-bound filter to exclude dormant contributors.

**Key quote**
> "I will not engage with AI slop. [...] I don't have time to waste on it." — brian m. carlson, on AI-generated review responses

**Next steps**
The discussion now centers on:
1. **Eligibility criteria**: Balancing inclusivity, recency, and impact (e.g., "two or more topics in a released version of Git").
2. **Voting mechanism**: Whether to use Debian's *devotee*, GitHub polls, or an email-based system.
3. **Secret ballots**: How to reconcile anonymity with the project's transparency norms.

---

### Performance: Incremental connectivity checks for fetch/push
**What's changing?**
Kristofer Karlsson's six-patch series introduces an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental` or `auto`) to reduce the cost of verifying object connectivity during fetches and pushes. The new mode processes incoming commits in topological order, shifting cost from repository size to incoming change size.

**Why it matters**
The existing full check scales poorly with repository size, making fetches and pushes slow in large monorepos. The incremental mode is up to **190× faster** for small pushes to large repos (14 MB vs. 3.4 GB memory) and **2.6–2.8× faster** for large/medium repos at 1,000 commits.

**Today's developments**
- **Forward-compatibility resolved**: Kristofer Karlsson agreed to eliminate reliance on `repo->fetch_if_missing` by switching to `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT`, addressing Junio's concern about lazy-fetching assumptions.
- **Junio requested** a commit-message tweak to clarify the lazy-fetching behavior, but Kristofer confirmed the tweak is now unnecessary as the code no longer touches `fetch_if_missing`.

**Key quote**
> "This is exciting." — Junio C Hamano, on the series' potential

**Next steps**
The series is **ready for integration**, pending Junio's final sign-off. The only remaining action is to address the minor commit-message clarification, which may be moot after the latest changes.

---

### Bugfix: SSH signing verification flaws
**What's changing?**
Christian Noé Ramos López fixed two critical flaws in Git's SSH signing verification:
1. **Timestamp confusion**: `valid-before` checks used the committer date (signer-controlled) instead of the signature timestamp, allowing backdated commits to bypass key expiration.
2. **Revocation file fails open**: If `gpg.ssh.revocationFile` was configured but missing, Git warned and proceeded as if no revocation file existed, instead of failing closed.

**Why it matters**
These flaws undermined the security guarantees of SSH signing, particularly for teams using key expiration or revocation lists to manage access.

**Today's developments**
- **Documentation updated**: The `gpg.adoc` man page now clarifies that Git checks the *committer or tagger date* (not the signature timestamp) against the key's validity range, and explicitly notes the lack of security guarantees due to signer-controlled timestamps.
- **v2 security bugfix patch**: The revocation file flaw is now fixed by moving the missing-file check into the same block that handles unknown keys, ensuring signature information is displayed consistently. Junio requested that future rerolls be sent as direct replies to maintain threading for automation.

**Key quote**
> "This does not provide any security guarantees as the timestamp can be chosen arbitrarily by the signer." — Patrick Steinhardt, on the timestamp confusion flaw

**Next steps**
Both patches are **ready for integration**, though they may be held together to avoid splitting security fixes across releases.

---

### Performance: `git log --follow` optimization for merges
**What's changing?**
Miklos Vajna's four-patch series fixes a severe performance regression in `git log --follow` introduced by commit `304812ed33` (June 2026). The regression made `--follow` unusably slow (6m43s vs. 28s on the Linux kernel repository) when tracking renames across merges.

**Why it matters**
`git log --follow` is a critical tool for understanding file history, and the regression made it impractical for large repositories.

**Today's developments**
- **v3 series posted**: The final patch splits diffing into two passes: a fast pass without rename detection to identify parents where the path is unchanged, and a fallback to the expensive rename-detection pass only if the path differs in *all* parents. This restores performance to near the pre-regression baseline (36s vs. 28s).
- **Test coverage expanded**: The first patch adds extensive test coverage for `--follow` behavior across merges, including edge cases with renames on both sides of a fork.

**Key quote**
> "The performance regression is so severe that it makes `--follow` unusable for large repositories." — Miklos Vajna, in the commit message

**Next steps**
The series is **under review** and appears ready for `next` unless new concerns emerge about the two-pass diffing approach.

---

### Bugfix: Index locking behavior in `git status`
**What's changing?**
Mihai Cosma's two-patch series addresses two issues in `git status`:
1. **Silent data-loss bug**: In the `index.skipHash` code path (enabled in Scalar repos and with `feature.manyFiles`), `git status` could overwrite concurrent index modifications without detection.
2. **Lock contention**: `git status` held `index.lock` for its entire duration, blocking concurrent operations like `git add` or `git commit`.

**Why it matters**
The data-loss bug affects large repositories using Scalar or `feature.manyFiles`, while the lock contention issue disrupts IDE workflows (e.g., background `git status` calls blocking user-initiated commands).

**Today's developments**
- **Benchmark results**: In a large repository (gcc, 124k files), concurrent `git add` operations went from failing **50-60% of the time** to near-zero failures, with a minor latency trade-off (~128ms vs. ~114ms in the clean case).
- **Alternative approach**: Mihai also proposed modifying `git status` to acquire the index lock only when necessary (e.g., for untracked cache updates), reporting a **6× speedup** in `git add` latency (from 1020 ms to 160 ms) in a gcc checkout with background `status` loops.

**Key quote**
> "The current behavior is actively harmful in large repositories." — Mihai Cosma, on lock contention

**Next steps**
The series is **under review** and may interact with Jun Wu's concurrent "Wait for the index lock" series. The author has indicated willingness to rebase if needed.

---

## In brief
- **`git worktree --reflink` and ephemeral branch reaping**: brian m. carlson objected to Jason Newton's use of AI-generated text in review responses, escalating the interpersonal conflict in the thread. The **Windows ReFS reflink** remains permanently disabled due to silent corruption risks.
- **`gitbreaking-changes(7)` manpage**: Junio signaled tentative approval for the series, contingent on the post-transition awkwardness being manageable. The document's lifecycle after Git 3.0 is now resolved in principle (living draft or retain only the most recent entries).
- **`git branch --delete-merged` squash/rebase detection**: The CLI redesign discussion converged on `--delete-integrated <pattern>` (change-based detection for rebased *and* squashed branches) with no config variable for intermediate levels. Jasper Mayone proposed an **author-check mitigation** to reduce false positives.
- **`git stash` performance**: Phillip Wood's series to eliminate redundant `check_changes()` calls is **queued for `next`**, with a v4 planned to address stylistic tweaks.
- **Shallow fetch regression**: Karthik Nayak posted a v4 patch exposing already-fetched refs to negotiation via `negotiation_include_tips`, restoring correct 'have' reporting while preserving atomicity for `--atomic` fetches.
- **`git remote set-password`**: brian m. carlson **rejected the proposal outright**, citing project policy against encouraging URL-embedded credentials and advocating for credential helpers or SSH with public keys.
- **`git init` absolute paths**: Matt Hunter posted a v2 patch restoring the pre-20e38cba3926 behavior of printing the absolute path in the "Initialized empty Git repository" message.
- **Terminology cleanup**: Andy Koppe's five-patch series aligns Git's documentation, code comments, and tests with the 2024 redefinition of "pseudoref" (now limited to FETCH_HEAD and MERGE_HEAD) and "root ref" (refs outside `refs/`). Junio praised the series as "a pleasant read."
- **Outreachy contributions**: Several microprojects progressed, including test modernizations, i18n pluralization fixes, and a bugfix for suppressed exit codes in `t/t2104-update-index-skip-worktree.sh`.

---

## Key decisions and actions
1. **Governance**: The voting process proposal will proceed with refinements to eligibility criteria and secret ballots. Emily Shaffer plans to self-review the patch to flag open questions.
2. **Incremental connectivity checks**: Junio to pick up the series for integration once the minor commit-message clarification is addressed.
3. **SSH signing verification**: Both patches are ready for integration, pending Junio's queueing.
4. **`git log --follow` optimization**: The series is under review; no blockers identified yet.
5. **Index locking**: Mihai's series is under review; Junio to evaluate the trade-offs between the timeout approach (Jun Wu) and the root-cause fix (Mihai).