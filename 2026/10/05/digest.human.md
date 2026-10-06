# Git mailing list daily digest for 2026/10/05

## The day in brief
The Git mailing list saw **94 emails** today, with key developments in **signing rewritten commits**, **SHA-256 transition debates**, **shallow clone safeguards**, and **CI leak sanitizer improvements**. The most consequential exchanges centered on **Scott Chacon’s challenge to the Git 3.0 SHA-256 transition plan**, **Patrick Steinhardt’s proposal to warn users about destructive operations on shallow clones**, and **Harald Nordgren’s compromise for GitHub Actions leak sanitizer failure reporting**.

---

## Notable threads

### 1. Teach `git history` to sign rewritten commits
**What changed?**
Souma’s patch series (v5) to teach `git history` to sign rewritten commits (`drop`, `fixup`, `reword`, `split`, `squash`) using GPG is now **unblocked** after resolving a completion test regression. The series is **cooking in `seen`** and ready to graduate to `next`.

### Why it matters

This feature aligns `git history` with other Git commands (`commit`, `rebase`, `cherry-pick`) that already support signing, improving security for interactive history rewriting workflows. The completion fix ensures users can tab-complete `--gpg-sign` and `--no-gpg-sign` options, matching `git rebase` and `git commit` behavior.

### Key details

- **Files touched**: `builtin/history.c`, `replay.c`, `Documentation/git-history.adoc`, `t/t3451-history-reword.sh`, `t/t9902-completion.sh`
- **New behavior**: All five subcommands respect `commit.gpgsign`, `-S[=<key-id>]`, `--gpg-sign[=<key-id>]`, and `--no-gpg-sign`; command-line options override configuration.
- **Completion protocol**: `--git-completion-helper` for `git history split` lists static options before `--` and generated negated options after it (e.g., `--no-gpg-sign`).

### Today’s delta

- [Souma] Clarified that `...` in the completion test is a **literal placeholder** for dynamically generated `--no-*` options, resolving Junio’s question.

---

### 2. Git 3.0 SHA-256 transition: strategic debate
**What changed?**
Scott Chacon **escalated the debate** over Git 3.0’s SHA-256 transition plan, arguing it is a **GitHub-driven effort lacking broad community buy-in** and proposing his tree-digest signing mechanism as a less disruptive alternative. Brian M. Carlson and Patrick Steinhardt countered that the transition is **necessary for regulatory compliance and ecosystem momentum**.

### Why it matters

The discussion now centers on **project governance** and whether the Git 3.0 timeline (March 2027) reflects genuine community consensus or corporate priorities. Chacon’s proposal—embedding SHA-256 digests of tree contents in signed commits/tags—could satisfy signature security requirements without switching the default object format, but Carlson insists the full transition is non-negotiable.

### Key details

- **Chacon’s argument**: SHA-1’s content-addressing role does not require collision resistance; the tree-digest mechanism satisfies regulatory mandates for signatures.
- **Carlson’s rebuttal**: The transition is needed for security researchers to store colliding blobs and for institutional adoption.
- **Steinhardt’s warning**: Abandoning the transition would **ossify the ecosystem**, making future breaking changes exponentially harder.

### Today’s deltas

- [Chacon] Framed the SHA-256 effort as **GitHub-driven**, citing internal prioritization conflicts.
- [Carlson] Pushed back, emphasizing **personal contributions** and institutional demand.
- [Steinhardt] Warned that **delay would kill momentum**, as the Git 3.0 announcement was the only reason major forges began implementing SHA-256 support.

---

### 3. Safeguards for shallow clone destructive operations
**What changed?**
Patrick Steinhardt proposed that Git **warn or refuse by default** when users attempt destructive operations (e.g., `git revert`, `git commit --amend`) on shallow boundary commits, citing surprising behavior that can leave the working tree empty. Junio C Hamano clarified that shallow clones **preserve original commit IDs** but logically rewrite boundary commits as root commits during traversal.

### Why it matters

Shallow clones (`--depth=1`) are widely used in CI/CD and large repositories, but their behavior during destructive operations can be **confusing and potentially destructive**. Steinhardt’s proposal aims to prevent unintended data loss, while Junio’s clarification reframes the architectural challenge as one of **runtime traversal logic**.

### Key details

- **Problem**: Reverting a shallow boundary commit removes all tracked files, leaving an empty working tree.
- **Proposal**: Warn or refuse operations on shallow boundary commits, with an override mechanism (e.g., `--force`).
- **Architectural insight**: Shallow clones use a **grafts/replace-like mechanism** to hide true parents during traversal, but commit objects retain their original IDs.

### Today’s deltas

- [Steinhardt] Argued the current behavior is **surprising and likely unintended**, proposing safeguards.
- [Junio] Clarified that the **runtime illusion of root commits** is the root cause, not object identity.
- [Sphinx] Agreed with Steinhardt’s framing, signaling support for safeguards.

---

### 4. GitHub Actions leak sanitizer failure reporting
**What changed?**
Harald Nordgren proposed a **compromise** for the abandoned second patch in his leak sanitizer failure reporting series. Instead of linking annotations to test-script lines (which broke when the test file was unmodified), the compromise **embeds the file name and line number directly in the annotation text** while preserving job-output links.

### Why it matters

The original patch’s annotation links were **useless or broken** unless the test file was modified in the same pull request, creating a regression. The compromise avoids this while still providing more context than the current behavior, improving the usability of GitHub Actions CI for Git.

### Key details

- **Files touched**: `ci/lib.sh`, `t/test-lib-github-workflow-markup.sh`
- **New behavior**: Annotations now include file/line context (e.g., `memory leak logged in t1060 (t1060-object-corruption.sh)`).
- **Compromise**: Preserves job-output links while embedding context in the annotation text.

### Today’s deltas

- [Phillip Wood] Formally recommended **dropping the second patch** due to broken links.
- [Harald Nordgren] Proposed the **compromise**, which Phillip accepted.

---

### 5. `git repo structure` revision-based filtering
**What changed?**
Mark C. Chu-Carroll **abandoned the `--no-<reftype>` approach** in favor of a **revision-based filtering system** for `git repo structure`, using Git’s standard `setup_revisions()` machinery. The new design allows users to specify arbitrary revisions (e.g., `git repo structure --branches --not --tags` or `git repo structure master`).

### Why it matters

The original design was **inflexible and inconsistent** with Git’s idioms. The new approach is **strictly more expressive**, avoiding the transitive dependency omission issue of v1 by focusing on reachability rather than reference types.

### Key details

- **Files touched**: `builtin/repo.c`, `Documentation/git-repo.adoc`, `t/t1901-repo-structure.sh`
- **New behavior**: Users can filter by any revision, not just predefined reference types.
- **Performance**: Reachability logic ensures all objects reachable from included revisions are processed unless they are exclusively reachable via excluded revisions.

### Today’s deltas

- [Mark C. Chu-Carroll] Implemented the **revision-based filtering system**, addressing Patrick Steinhardt’s critique of v1.

---

## In brief
- **`git blame` ignore-revs default file**: Junio raised **security concerns** about parsing upstream-controlled content at a fixed path (`.git-blame-ignore-revs`), and Ravi Mistry proposed **security-focused refinements** (NUL byte rejection, promisor fetch prevention).
- **`git stash create` redesign**: Kazumasa Shigeta outlined **three directions** for exposing more of `do_create_stash()`’s capabilities, with Junio favoring `--create-only` for `git stash push`.
- **`git checkout -m` conflict labels**: Phillip Wood’s v2 series to preserve original conflict labels is **ready for integration**, pending resolution of the conflict style question.
- **`git fetch` incremental connectivity check**: Patrick Steinhardt raised **technical questions** about trust boundary detection and error handling, and Junio suggested **non-ref roots** (e.g., index entries) for connectivity checks.
- **Git for Windows 2.56.0(2)**: Johannes Schindelin released a **bugfix update** addressing regressions from the MINGW64 → UCRT64 migration.
- **`git repo` ref storage format**: Patrick Steinhardt renamed `references.format` to `references.storageFormat` for consistency.
- **ZIP timestamp bugs**: Matthew E. Luallen reported **three date-handling bugs** in Git’s ZIP archive generation and `git fast-import` logic, with René Scharfe proposing **clamping solutions**.

---

## Threads worth watching
- **SHA-256 transition debate**: The strategic discussion about Git 3.0’s timeline and Chacon’s tree-digest proposal is likely to continue, with Junio’s eventual response critical.
- **Shallow clone safeguards**: Steinhardt’s proposal to warn/refuse destructive operations on shallow boundary commits may evolve into a patch series.
- **`git stash create` redesign**: The discussion about exposing more of `do_create_stash()`’s capabilities is narrowing toward `--create-only` for `git stash push`.
- **ZIP timestamp bugs**: The reported issues in `git fast-import` and ZIP archive generation may prompt fixes for edge cases.