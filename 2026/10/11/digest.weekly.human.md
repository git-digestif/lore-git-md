# The Git Project -- Weekly Digest for 2026/10/05 -- 2026/10/11

## The period in brief

This week (2026/10/05--2026/10/11) saw **580 emails** across **six active days**, with traffic slightly above the seasonal average. The tone was **technically dense and policy-charged**: the Git 3.0 SHA-256 transition debate escalated into a governance flashpoint, while AI-assisted contributions sparked a multi-day discussion about project norms. Three developments stand out: **Scott Chacon’s challenge to the SHA-256 transition plan**, **the approval of `git blame`’s default ignore-revs file**, and **the finalization of the `rerere` race-condition fix**.

---

## Key developments

### Git 3.0 SHA-256 transition: strategic debate
Scott Chacon **escalated the debate** over Git 3.0’s SHA-256 transition plan, arguing it is a **GitHub-driven effort lacking broad community buy-in** and proposing his tree-digest signing mechanism as a less disruptive alternative. The discussion, which began on 2026/10/05, revealed deep divisions: **brian m. carlson** and **Patrick Steinhardt** countered that the transition is **necessary for regulatory compliance and ecosystem momentum**, while Chacon framed it as **corporate prioritization** that risks alienating the broader community.

By week’s end, the thread had expanded to include **Git Contributor’s Summit notes** (2026/10/06), which confirmed the two-stage rollout (2.99 → 3.0) but left unresolved questions about **LTS status for 2.99** and **Rust mandates**. The debate now centers on **project governance**: whether the Git 3.0 timeline reflects genuine community consensus or corporate priorities. Chacon’s tree-digest proposal—embedding SHA-256 digests of tree contents in signed commits/tags—remains a live alternative, though **no maintainer has endorsed it**.

### `git blame` default ignore-revs file lands
Ravi Mistry’s series teaching `git blame` to automatically use `HEAD:.git-blame-ignore-revs` as a default ignore-revs file **reached integration-ready status** after Junio C Hamano approved the v3 revision on 2026/10/11. The series addresses a long-standing usability gap by making ignore-revs files **discoverable and automatic**, while hardening the parser against malformed input (NUL bytes, promisor fetches in partial clones).

The implementation resolves the last maintainer request by replacing the global `the_repository` variable with `sb->repo` in `parse_default_ignore_revs_blob()`, ensuring thread safety. The series is now **cooking in `next`**, with no unresolved technical concerns.

### `rerere` race condition fix finalized
Thomas Bachem’s v7 series fixing a race condition in Git’s `rerere` subsystem **reached final form** after dropping the controversial third patch. The series introduces a configurable timeout (`rerere.lockTimeout`, defaulting to 1000 ms) for the `MERGE_RR.lock` file, allowing processes to wait briefly for the lock instead of failing immediately. Patch 2 adds a hidden `--skip-locked` flag to `git rerere gc`, enabling auto-maintenance tasks to skip silently if the lock is held.

The decision to drop the warning-and-proceed behavior reflects feedback from **Patrick Steinhardt**, who argued it was unnecessary without real-world user reports. Junio’s final review requested minor documentation and test improvements, which Bachem incorporated. The series is now **technically complete** and ready for integration.

### AI-assisted contributions: policy debate
Scott Chacon’s proposal to amend `Documentation/SubmittingPatches` to explicitly permit AI-assisted contributions **sparked a multi-day debate** about enforceability, legal risks, and reviewer bandwidth. The discussion, which began on 2026/10/07, revealed **three distinct positions**:
- **Chacon**: Proposes an `Assisted-by:` trailer for substantial AI assistance, requiring contributors to review, test, and sign off on submissions.
- **brian m. carlson**: Opposes the change on **legal and ethical grounds**, citing risks of license violations and copyright infringement from AI-generated code.
- **Junio C Hamano** and **Patrick Steinhardt**: Question **enforceability and reviewer bandwidth**, noting that the current policy ("we may reject anything that looks AI-generated") is more direct and easier to apply.

By week’s end, **no consensus had emerged**, but the discussion highlighted the tension between leveraging AI tools for productivity and maintaining the project’s review culture. **Maciej Ciemborowicz’s disclosure** (2026/10/08) of a seven-step AI-assisted workflow further underscored the moral and practical dilemmas facing contributors.

### ZIP archive timestamp bugfix
René Scharfe posted a patch addressing one of three date-handling bugs in Git’s ZIP export. The fix omits the Extended Timestamp (UT) extra field when the commit time exceeds the 32-bit signed integer range, preventing silent timestamp wrap that causes tools like UnZip to misbehave in update mode. The patch aligns with René’s updated position advocating **rejection (omission) of out-of-range timestamps** rather than clamping, prioritizing correctness over tool compatibility. Tests verify the UT field is included for 32-bit timestamps and omitted for out-of-range values.

---

## In brief

**`git history` signing** -- Souma’s v5 series teaching `git history` to sign rewritten commits (`drop`, `fixup`, `reword`, `split`, `squash`) using GPG **resolved its last blocking question** (completion test regression) and is now **cooking in `seen`**, ready to graduate to `next`.

**Shallow clone safeguards** -- Patrick Steinhardt’s proposal to warn or refuse destructive operations (e.g., `git revert`, `git commit --amend`) on shallow boundary commits **gained traction** after Junio C Hamano clarified the architectural root cause (runtime illusion of root commits). **Sphinx signaled support** for safeguards, though no patch series has been posted.

**GitHub Actions leak sanitizer** -- Harald Nordgren’s v6 series improving leak sanitizer failure reporting **reached a compromise** for broken annotation links, embedding file/line context directly in annotation text while preserving job-output links. The series is now a **candidate for `next`**.

**`git repo structure` revision filtering** -- Mark C. Chu-Carroll abandoned the `--no-<reftype>` approach in favor of a **revision-based filtering system**, using Git’s standard `setup_revisions()` machinery. The new design is **strictly more expressive**, avoiding the transitive dependency omission issue of v1.

**Global config handling** -- Delilah Ashley Wu’s v3 series fixing Git’s global config file handling inconsistency **reached technical completion**, aligning `--global` operations with unscoped behavior by reading both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`.

**`git push` refspec matching performance** -- Jon Simons’s 15-patch v1 series **achieved 10-100x speedups** in benchmarked scenarios by replacing linear ref traversals with `strmap` lookups. The improvements target large-repository workflows, particularly when pushing many branch deletions.

**`gitmergeconflicts(7)` guide** -- Julia Evans’s six-patch v2 series introducing `gitmergeconflicts(7)` **neared completion**, with Junio’s latest refinements incorporated into v3. The guide consolidates scattered advice on merge conflicts, improving clarity around "ours" vs. "theirs" and providing practical examples.

**Outreachy mentor reorganization** -- Git’s participation in the Outreachy December 2026 cohort was finalized with a mentor reorganization, approving three project ideas but limiting to two interns due to mentor availability.

**CI infrastructure debate** -- Patrick Steinhardt pushed back against Junio C Hamano’s proposal to remove the `linux32` and `linux-TEST-vars` jobs, arguing that i386 coverage and exotic configuration testing remain valuable.

**`git stash create` redesign** -- Kazumasa Shigeta and Phillip Wood narrowed the scope of proposed changes, ruling out `--patch` and positional pathspecs while endorsoring `--pathspec-from-file` and `-m`/`--message`.

**`git fetch --prune-tags` bugfix** -- Orgad Shaneh gently pinged Junio C Hamano for a substantive response on the series aligning implementation with documentation. The thread remains stalled pending Junio’s input.

**`git checkout -m` conflict labels** -- Phillip Wood’s v2 series to preserve original conflict labels **reached integration-ready status**, pending resolution of the conflict style question.

**`git repack` data-loss race fix** -- Qeesung’s v3 series fixing a data-loss race in `git repack -d` **addressed Junio’s layering concern** and is now blocked only by Patrick Steinhardt’s `ps/odb-files-alternates` topic.

**`git fast-import` crash fix** -- Aniket Gupta confirmed the fix resolves crashes for commits, tags, and tags-of-tags created mid-session. The patch adds a cache lookup in `parse_from_existing` to prevent crashes when resolving in-session objects.

**OpenSSL compatibility patch for `imap-send`** -- Xiangfei Ding posted a patch updating `imap-send.c` to stop assuming `ASN1_STRING` data is null-terminated, addressing an upcoming breaking change in OpenSSL derivatives like BoringSSL.

**`git worktree repair` validation** -- Yoichi NAKAYAMA agreed to split patch 1/2 into smaller commits to separate behavioral changes from refactoring, improving clarity for the worktree ID validation logic.

**`git branch --delete-merged` squash detection** -- Harald Nordgren posted v2 of the series, adding a `--no-squashed` flag to skip slower squash detection and fixing a merged-branch attribution bug.

**Test infrastructure cleanup** -- Junio C Hamano requested commit message clarification for the first patch in the mergesort Clar migration series.

**Documentation update** -- Jon Simons and Junio C Hamano discussed wording for a `SubmittingPatches` update discouraging the "add failing test, then fix" anti-pattern.

---

## Looking ahead

Three topics are likely to dominate next week:

1. **Git 3.0 SHA-256 transition**: The strategic debate will continue, with Junio’s eventual response critical to resolving the governance impasse. Chacon’s tree-digest proposal may gain traction if maintainers seek a compromise.
2. **AI-assisted contributions**: The policy discussion will likely evolve into a concrete patch or a decision to maintain the status quo. Contributors are watching closely for signals about how the project will handle AI-generated code.
3. **`git repo structure` revision filtering**: Mark C. Chu-Carroll’s v4 patch remains blocked by unaddressed feedback on commit message motivation and the double ref-walk inefficiency. A respin is expected.