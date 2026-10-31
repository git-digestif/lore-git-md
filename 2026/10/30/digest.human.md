# Git mailing list daily digest for 2026/10/30

## The day in brief

The Git project saw significant progress on several fronts today. The `git log --follow` optimization series (v4) was posted, dramatically improving performance for merge-heavy histories. A critical bug in SSH signing verification was identified, where `.gitattributes`-driven merge behavior could be silently ignored. The community voting process discussion advanced with a hybrid eligibility model proposed. The Rust reorganization patch remains in limbo due to lack of broader project enthusiasm. Several documentation improvements were merged, including a new `gitsecurityboundary(7)` man page.

## Notable threads

### Rust reorganization patch in limbo
**Thread root**: 2026/02/04/23-22-08

Mike Hommey reaffirmed his personal interest in the Rust reorganization patch (v5) that moves Rust files to a dedicated `rust/` subdirectory. The patch is technically complete and addresses all prior feedback, but Junio C Hamano has questioned whether the project wants to proceed at all, citing lack of broader enthusiasm and unresolved forward-compatibility concerns about Git 3.0's mandatory Rust components.

Mike's reply was a single-line statement: "I, obviously, am still interested. Selfishly, I'd rather avoid having to maintain a fork just to move files around." This reaffirms his stake in the outcome but provides no new technical arguments or advocacy to sway the decision. The patch remains queued for `next`, but Junio's earlier tone suggests it may be dropped unless someone else steps forward to advocate for it.

### What does the topic change?

The patch reorganizes Git's Rust infrastructure from `src/` to a dedicated `rust/` subdirectory to improve project structure clarity in Git's mixed C/Rust codebase.

### Problem/goal

The reorganization addresses immediate project hygiene concerns (misleading top-level Cargo.toml, mixed-language confusion) and was intended to establish a foundation for handling Git 3.0's mandatory Rust components.

### Subsystem

Rust infrastructure, build system

### Kind of change

Project structure reorganization

### Impact

Improves code organization and supports downstream projects that vendor Git's C code while excluding its Rust components. The patch is technically complete but lacks broader project consensus.

---

### SSH signing verification flaw: .gitattributes silently ignored
**Thread root**: 2026/09/29/18-51-22

Junio C Hamano identified a **potential correctness issue** in the squash-detection logic's handling of Git attributes. The `merge_keeps_upstream()` function in Harald Nordgren's `git branch --delete-integrated` series passes `the_repository->index` to `ll_merge()`, but the index may not be populated at this point. Since `ll_merge()` consults the index for custom merge drivers and attributes, passing an uninitialized `istate` could cause it to miss in-index attributes, leading to incorrect merge results or silent failures.

The issue affects the **path-by-path three-way merge approximation** introduced to handle upstream modifications in squash-detected branches. If attributes are not loaded, the merge may not respect `.gitattributes` directives, potentially causing false negatives (failing to recognize landed changes) or false positives (incorrectly marking changes as landed).

This is the first **substantive technical feedback** on the fourth patch of the series, which had previously been viewed as addressing the last major correctness gap. The issue is not a regression but a latent flaw in the new logic. Resolution will require either a code change (loading the index) or a documentation update (explicitly stating attributes are ignored).

### What does the topic change?

The patch implements squash-detection with upstream modifications via a path-by-path three-way merge approximation using `ll_merge()`.

### Problem/goal

The goal is to handle upstream modifications to squashed files, where the squash commit no longer contains the branch's exact file versions.

### Subsystem

Branch management, merge machinery

### Kind of change

Bugfix and feature enhancement

### Impact

The correctness issue could lead to incorrect branch deletion decisions when upstream modifications are present. The fix is essential for the safety of the `--delete-integrated` feature.

---

### Community voting process: hybrid eligibility model proposed
**Thread root**: 2026/10/06/18-05-36

Jeff King (Peff) proposed a **hybrid eligibility model** for the Git project's community voting process, combining an algorithmic threshold with a human-curated list for edge cases. This model reconciles the tension between Phillip Wood's call for a "middle ground" and Emily Shaffer's advocacy for broader inclusivity, while keeping administrative overhead manageable.

The hybrid model would:
1. Define a quantitative rule (e.g., "X commits and active in the last Y years") for most contributors
2. Supplement this with a human-curated list for edge cases (e.g., former maintainers, deeply involved but inactive contributors)

Peff volunteered Junio C Hamano or the Project Leadership Committee (PLC) as arbiters of the human-curated list, suggesting a one-time "big initial list" with periodic updates. This model addresses Emily's concern about excluding long-term contributors while keeping administrative overhead manageable.

The project must now decide:
- Whether to adopt this hybrid model, Junio's broader approach, or another solution
- How to operationalize the algorithmic threshold (e.g., commit count, time window, patch series vs. individual commits)
- Who will maintain the human-curated list and how often it will be updated

Jonathan Corbet provided a **substantive, tested review** of CIVS (Condorcet Internet Voting Service), confirming its practicality for occasional polls but highlighting a **usability hurdle**: voters must proactively sign up on the platform, which could create friction for Git's email-centric workflow.

### What does the topic change?

The community voting process is a formal mechanism to resolve contentious decisions when consensus cannot be reached through normal discussion.

### Problem/goal

The goal is to establish governance mechanisms for divisive issues like the AI contribution policy.

### Subsystem

Project governance

### Kind of change

Process and documentation

### Impact

The voting process will determine how contentious decisions are made in the Git project, with significant implications for the project's direction and inclusivity.

---

### `git log --follow` optimization for merges
**Thread root**: 2026/10/30/20-07-00

Miklos Vajna posted a **v4 series** that fixes bugs and optimizes `git log --follow` for merges. The series addresses a performance regression introduced in Git 2.56.0 (commit 304812ed33) that made `git log --follow` significantly slower when processing merges.

The series:
1. Fixes a correctness bug where `-m --follow` loses history from non-first parents
2. Fixes a bug where `--follow` misses edits on dropped sides of merges
3. Introduces a targeted optimization that reduces the number of rename-detection passes needed when following a file through merges

The optimization cuts runtime for a real-world linux.git case from ~3 minutes to ~37 seconds. The series is technically complete, with all known bugs fixed and performance regressions addressed. The interdiff shows the key technical change: the rename-detection logic now only checks for renames from files that the merge actually deleted, rather than scanning all files in every parent.

### What does the topic change?

The series fixes correctness bugs and optimizes the `git log --follow` command for non-linear history (merges).

### Problem/goal

The goal is to fix performance regressions and correctness issues in `git log --follow` when dealing with merges.

### Subsystem

Revision walking, diff machinery

### Kind of change

Bugfix and performance optimization

### Impact

The optimization dramatically improves performance for users working with large repositories and complex merge histories. The correctness fixes ensure that `git log --follow` provides accurate results in all scenarios.

---

### Documentation improvements merged
**Thread root**: 2026/10/06/19-37-00

Julia Evans's documentation series replacing the outdated `gittutorial` man page with a beginner-focused guide was **merged to `next`**. The series is now mechanically complete and ready for integration testing, with all substantive and surface-level feedback addressed.

The new tutorial is user-tested with 112 participants and designed to be completed in under an hour. It covers local repository creation and pushing to a remote host, with platform-specific authentication instructions (SSH for macOS/Linux, Git Credential Manager for Windows). The tutorial avoids explaining branches, deferring to a future guide, and includes a troubleshooting section for common pitfalls.

Junio noted that while the series was merged, there are still unresolved mechanical issues affecting `make check-docs`. He plans to address these with follow-up updates but did not treat them as blockers.

### What does the topic change?

The series replaces the outdated `gittutorial` man page with a modern, beginner-focused guide.

### Problem/goal

The goal is to provide a practical introduction to Git that reflects current workflows while avoiding confusion about its decentralized model.

### Subsystem

Documentation

### Kind of change

Documentation improvement

### Impact

The new tutorial improves the onboarding experience for new Git users, making the project more accessible to beginners.

---

## In brief

- **`git grep --recurse-submodules` regression fix**: Patrick Steinhardt posted a series to fix a regression where `git grep --recurse-submodules` fails when grepping nested submodule trees. The fix ensures objects are read from the correct repository instance. (2026/10/30/10-21-46)
- **SSH signing verification flaws**: Christian Noé Ramos López posted the final version of the documentation patch clarifying how Git checks SSH key lifetimes. The patch explicitly states that Git uses the committer or tagger date for `valid-before`/`valid-after` checks and warns that this provides no security guarantees. (2026/10/30/06-15-28)
- **`gitsecurityboundary(7)` man page**: Johannes Schindelin introduced a new man page to define Git's security boundary, reducing misaligned security reports. The document clarifies that Git's security guarantees cover only unexpected or implicit execution of attacker-controlled code. (2026/10/30/12-12-50)
- **64-to-32 truncation fixes**: Elia Pinto posted a series fixing 64-to-32 truncations on LLP64 platforms (e.g., Windows), where `long` is 32-bit but `size_t` is 64-bit. The series addresses truncation/overflow issues in object database writes, word-diff buffers, file conversion helpers, and text statistics counters. (2026/10/30/09-58-49)
- **Worktree safety improvements**: Jasper Mayone proposed a series to make `git worktree remove` refuse deletion of in-progress worktrees and extend `%(worktreepath)` to include rebasing/bisecting branches. The series addresses safety concerns raised in the RFC discussion. (2026/10/30/14-21-35)
- **Loose object handling fixes**: Elia Pinto posted a series fixing bugs in loose object handling, including missing size checks and truncation issues. The series hardens size-related code paths against truncation and overflow. (2026/10/30/10-28-33)
- **`.gitignore` documentation clarification**: Junio C Hamano proposed a rewrite of the `gitignore(5)` man page to explicitly state that the `*` wildcard matches strings of any length, including zero. This addresses a user question about pattern matching behavior. (2026/10/30/16-59-12)
- **Function encapsulation cleanup**: Junio C Hamano posted a series marking functions as `static` where they are no longer needed outside their source files. The series improves code hygiene by reducing global symbol exposure. (2026/10/30/17-39-52)
- **`the_repository` removal**: Okhuomon Ajayi posted a patch replacing `the_repository` with `istate->repo` in three functions in `name-hash.c`, removing reliance on the global variable. This is part of the ongoing effort to eliminate `the_repository`. (2026/10/30/18-32-18)