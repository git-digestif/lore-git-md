# Git mailing list daily digest for 2026/10/24

## The day in brief
The Git mailing list saw active discussion across several key areas today. The most consequential developments include: a critical bugfix for the `remote.<name>.refmap` series that resolves a fallback logic issue in `remote_find_tracking()`; a philosophical debate about Git's defaults for triangular workflows reaching new clarity; and multiple refactoring efforts advancing the libification and `the_repository` removal initiatives. The community also made progress on several long-standing feature discussions, including the reflog usability series and the community voting process RFC.

## Notable threads

### [PATCH] fetch: introduce `+:` refspec for matching-inspired fetch mode
**What changed**: Harald Nordgren submitted a bugfix for the v10 series introducing `remote.<name>.refmap`, addressing a technical concern raised by D. Ben Knoble about the fallback logic in `remote_find_tracking()`. The fix ensures the function only falls back to `remote->refmap` when `remote->fetch` is either empty or contains no matching refspec for the given query.

**Why it matters**: This series aims to improve fetch performance for shallow repositories by dynamically fetching only branches tracked by local branches. The bugfix resolves a potential edge case where the fallback logic could return unintended refspecs, affecting downstream operations like `--delete-merged`. The series is now technically complete and ready for merging to `next` once the dependency (`hn/fetch-refmap`) graduates.

### Key details

- The fix replaces a two-branch `if/else` with a single condition that checks both the presence of `remote->fetch` refspecs and the success of `refspec_find_match()`.
- The series introduces a new configuration mechanism (`remote.<name>.refmap`) and a `--limited-fetch` flag for `git remote add`, with shallow repositories defaulting to this mode.
- Files touched: `remote.c`, `builtin/fetch.c`, `builtin/remote.c`, documentation, and tests.

---

### Git Contributor's Summit 2026: Community voting process eligibility criteria
**What changed**: The ongoing debate about eligibility criteria for Git's proposed community voting process gained new perspectives. Tuomas Ahola surfaced a discrepancy between Peff's original suggestion (50 merged patches) and current proposals (e.g., Junio's five commits), while Weijie Yuan expressed discomfort with a low threshold potentially allowing less-experienced contributors to influence high-stakes decisions. Junio refined his critique, distinguishing between the nature of changes in `@{upstream}` (external surprises) versus `@{push}` (self-authored work).

**Why it matters**: This discussion shapes Git's governance model for resolving contentious decisions (e.g., AI contribution policy). The thread now centers on whether to prioritize inclusivity (lower threshold) or ensure voters have a meaningful track record (higher threshold). The outcome will determine how Git balances community representation with technical rigor in future votes.

### Key details

- **Positions**:
  - Junio: `@{push}` divergence is less critical to monitor by default because it reflects the user's own work.
  - Weijie Yuan: Expressed hesitation about participating in votes where less-experienced contributors could influence decisions.
  - D. Ben Knoble: Provided anecdotal evidence of workflows where monitoring `@{push}` divergence is valuable.
- **Open questions**: Whether to raise the threshold to better reflect the summit's original intent, or accept a lower bar for practicality.

---

### [PATCH 0/3] reflog: add --operations and --historical-branch options
**What changed**: The reflog usability series took a significant turn after Phillip Wood's substantive review. Harald Nordgren tentatively accepted Phillip's proposal to replace `--historical-branch` with a format specifier (`%gb`), which would also drop `--operations` from the series. Junio proposed a broader refactoring of multi-step commands (e.g., `git am`, `git cherry-pick`) to work on detached HEAD by default, which would naturally produce cleaner reflog entries.

**Why it matters**: The series aims to improve reflog usability by collapsing multi-step operations into single entries and showing branch context. The shift to format specifiers aligns with Git's existing decoration system and addresses a critical correctness gap in `--historical-branch`'s handling of `git rebase <upstream> <branch>`. The discussion now hinges on whether Phillip's format-specifier approach can be implemented robustly.

### Key details

- **Critiques**:
  - `--operations` oversells its capabilities (not all commands update the reflog, e.g., `git cherry-pick A B C`).
  - `--historical-branch` has a correctness gap in handling `git rebase <upstream> <branch>`.
- **Proposal**: Replace `--historical-branch` with `%gb` to allow users to include branch names in custom formats.
- **Files touched**: `reflog-walk.c`, `revision.c`, `revision.h`, `log-tree.c`, `builtin/rebase.c`, `builtin/checkout.c`, `reflog.h`, documentation, and tests.

---

### [PATCH] status: default to comparing against both @{upstream} and @{push}
**What changed**: The debate over `git status` defaults for triangular workflows deepened with new user-centric arguments. D. Ben Knoble described his workflow, where he frequently checks divergence between local and `@{push}` branches to answer questions like *"Have I rebased lately and need to force-push?"*. Junio refined his critique, arguing that `@{push}` divergence is less critical to monitor by default because it reflects the user's own work.

**Why it matters**: This patch targets users of triangular workflows (e.g., GitHub PRs), where the push target (`@{push}`) differs from the merge target (`@{upstream}`). The discussion now centers on whether `@{push}` divergence is inherently less important than `@{upstream}` divergence, with implications for Git's default behavior in such workflows.

### Key details

- **Positions**:
  - Junio: `@{push}` divergence is less critical because it reflects the user's own work.
  - D. Ben Knoble: Provided anecdotal evidence of workflows where monitoring `@{push}` divergence is valuable (e.g., force-push decisions).
- **Patch status**: Under review; v2 addresses prior feedback but reopens the justification debate.

---

### [PATCH] remote: defer validation of remote.<name>.followRemoteHEAD
**What changed**: Junio C Hamano reported a mechanical conflict with Harald Nordgren's in-flight `hn/fetch-refmap` topic, which is blocking integration into `seen`. Matt Hunter's explicit approval ("Looks good") signals the patch is otherwise ready for `next`.

**Why it matters**: This patch eliminates spurious warnings when `remote.<name>.followRemoteHEAD` is loaded by commands that never use the setting (e.g., `git push`, `git remote -v`). It aligns with an earlier change (86129fd) for `fetch.followRemoteHEAD` and is part of a broader effort to reduce unnecessary noise in Git's output.

### Key details

- **Conflict**: Mechanical overlap with `hn/fetch-refmap` in `remote.c` or `fetch.c`.
- **Files touched**: `remote.c`, `builtin/fetch.c`, `builtin/remote.c`, documentation, and tests.
- **Next steps**: Rebase on top of `hn/fetch-refmap` once it graduates to `next` or `master`.

---

### [PATCH] extensions.objectStorage: pluggable object database backend
**What changed**: SZEDER Gábor raised a fundamental design question about whether the object storage backend should be user-configurable at all. The review challenges the motivation behind exposing `extensions.objectStorage` as a public API, shifting the thread's focus from mechanical correctness to architectural scope.

**Why it matters**: This series enables alternative object storage backends (e.g., cloud storage, databases) by making Git's object database pluggable. The question of whether user-facing configurability is necessary could influence the design of future pluggable subsystems in Git.

### Key details

- **Review**: SZEDER Gábor questions whether `extensions.objectStorage` should exist as a public API, given that the `files` backend's payload only relocates `.git/objects`.
- **Files touched**: `odb/`, `setup.c`, `repository.c`, `repository.h`, `builtin/`, documentation, and tests.
- **Open question**: Is user-facing configurability necessary for pluggable backends, or should backend selection be internal to Git?

---

### [PATCH] diff-delta: use safe integer helpers for allocation size calculations
**What changed**: Junio C Hamano requested a clearer explanation in the commit message of the theoretical overflow risk, specifically asking for a step-by-step analysis connecting the bounds on `entries` and `hsize`, struct sizes, and how padding could trigger overflow. Srujith Poojary posted v4 with an expanded commit message addressing this feedback.

**Why it matters**: This patch is part of a broader effort to adopt safe integer helpers across the codebase. The outcome may set a precedent for how rigorously the project evaluates theoretical fixes for unobserved risks.

### Key details

- **Files touched**: `diff-delta.c`.
- **Risk**: Theoretical overflow under 8-byte struct padding on 32-bit platforms.
- **Next steps**: Junio to assess the updated justification in v4.

---

### [PATCH] object-name: fix pluralization in reflog error message
**What changed**: Junio C Hamano provided a concrete patch to update `t1506-rev-parse-diagnosis.sh` to match the new behavior, replacing the old regex pattern with a more flexible one that works for both singular and plural cases. The v2 patch is now ready for review.

**Why it matters**: This i18n improvement fixes a pluralization error in the reflog lookup error message (e.g., "only has 1 entries" → "only has 1 entry") and enables proper plural forms in translations. It is a small but user-visible correctness fix.

### Key details

- **Files touched**: `object-name.c`, `t1400-update-ref.sh`, `t1506-rev-parse-diagnosis.sh`.
- **New behavior**: Uses "entry" for count=1, "entries" for count>1.

---

### [PATCH] reflog: fix dry-run output inconsistency for named refs
**What changed**: Laveesh Bansal submitted a bugfix for a UI inconsistency in `git reflog expire --dry-run --verbose`: when `--all` is used, the output says "would prune" for entries that would be removed, but when specific refs are named, it incorrectly says "prune" instead of "would prune".

**Why it matters**: This patch ensures consistent output wording during dry runs, improving usability and reducing confusion for users who rely on `git reflog expire --dry-run` to preview changes.

### Key details

- **Files touched**: `builtin/reflog.c`, `t/t1419-reflog-expire.sh`.
- **New behavior**: `--dry-run --verbose` now consistently uses "would prune" for all refs.

---

### [PATCH] completion: remove NO_PARSEOPT for commands using parse-options
**What changed**: Tanga Joris submitted a small but user-visible fix to Git's command-line completion. The patch removes the `NO_PARSEOPT` flag from four built-in commands (`patch-id`, `diff-pairs`, `fast-import`, and `var`) that have already been converted to use Git's `parse-options` API but were still incorrectly marked as hand-parsing their options.

**Why it matters**: This fix ensures Bash completion correctly offers options for these commands (e.g., `git fast-import --<TAB>` will now show available flags instead of nothing), improving the user experience for these less commonly used commands.

### Key details

- **Files touched**: `git.c`.
- **New behavior**: Bash completion now shows available options for the four commands.

---

### [PATCH] gitk: add "Cherry-pick with -x" context-menu action
**What changed**: Johannes Sixt (Hannes) reviewed the patch, questioning the necessity of the feature and suggesting UI/implementation refinements. He expressed hesitation about cluttering the already-long context menu with a variant of an existing command and invited broader community input.

**Why it matters**: This patch adds a context-menu entry in `gitk` that invokes `git cherry-pick -x` on the selected commit, appending the “(cherry picked from …)” trailer. The discussion highlights the tension between adding convenience features and avoiding menu clutter in `gitk`.

### Key details

- **Files touched**: `gitk-git/gitk`.
- **Review feedback**:
  - Menu label should avoid jargon ("-x") and use "Cherry-pick with cross-reference".
  - The Tcl proc `cherrypick` should default its optional parameter to `-r` (no trailer) rather than `-x`.
  - Translation files should not be manually edited.

---

### [PATCH v2] advice: add scope hint for disabling advice messages
**What changed**: Junio C Hamano marked the `jc/advice-config-set-global` topic (v5) for the `next` integration branch after Jeff King endorsed the core design and Vsevolod Myalitsyn agreed to defer the `vadvise()` interface cleanup to a follow-up patch.

**Why it matters**: This patch adds a scope hint to Git's advice settings, ensuring that when users are told how to disable an advice message (e.g., `git config advice.detachedHead false`), the command includes the correct config scope (`--global`). The series has converged on a uniform `--global` hint for all `advice.*` settings, simplifying the design.

### Key details

- **Files touched**: `advice.c`, twelve test scripts.
- **New behavior**: Always recommends `--global` in the hint for all `advice.*` settings.

---

## In brief
- **[PATCH] ci: pin GitHub Actions to full commit IDs**: Taylor Blau defended the security rationale for pinning GitHub Actions to full commit SHAs, arguing it provides meaningful security benefits by making CI dependencies explicit and verifiable. Junio C Hamano conditionally accepted the change pending a revised commit message.
- **[PATCH] What's cooking in git.git (Oct 2026, #06)**: Torsten Bögershausen submitted a platform-compatibility bugfix for a test in the recently merged `ps/pack-check-bigfile-threshold` topic, addressing a macOS-specific failure in `t/t1450-fsck.sh` where the BSD `mv` utility prompts for confirmation before overwriting a read-only file.
- **RFC: `git worktree list --merged` / `--no-merged`**: Eric Sunshine proposed a `for-each`-style mechanism akin to `git submodule foreach` as a more general alternative to the `--merged`/`--no-merged` proposal. Jasper Mayone implemented `--branch=<branch>` filtering for `git worktree list`, following Junio C Hamano's suggestion.
- **SHA-1/SHA-256 interoperability: relax unknown commit header handling**: Adam Majer clarified that missing `mergetag` objects in the Linux kernel repository often point to external repositories (some now defunct or unreachable), arguing that treating these headers as opaque text during conversion is the only way to preserve history deterministically.
- **Test coverage for `git check-ref-format --no-allow-onelevel`**: Phillip Wood endorsed the test coverage gap for `git check-ref-format --no-allow-onelevel` as a suitable Outreachy microproject, noting its small scope and clear benefit.
- **test modernization: t1006-cat-file.sh and %(rest) whitespace handling**: jyotish kumar proposed a test modernization for `t1006-cat-file.sh`, addressing a long-standing `FIXME` where the `%(rest)` format atom in `git cat-file --batch-check` fails when the object name contains whitespace.
- **regression: BUG() in `should_include_obj()` during `--filter=tree:0` + `--unshallow` fetch**: Daniel Zoghalchali reported a regression in Git 2.56.0 where a fetch combining `--filter=tree:0` with `--unshallow` aborts with a BUG() in `should_include_obj()` when the fetched ref has moved past the local shallow tip.
- **oss-fuzz regression in `parse_commit_graph()` after repository parameter refactoring**: Sami Jawhar submitted a bugfix for the oss-fuzz integration, addressing a regression introduced in commit f1141b4391 (2025-08-15) where `parse_commit_graph()` was refactored to take a repository parameter and now calls `prepare_repo_settings()`.
- **[PATCH] reflog: fix dry-run output inconsistency for named refs**: Laveesh Bansal submitted a bugfix for a UI inconsistency in `git reflog expire --dry-run --verbose`, ensuring consistent output wording during dry runs.
- **[PATCH] object-name: fix pluralization in reflog error message**: Aniket Gupta posted v2 of the i18n bugfix patch for the reflog error message pluralization, updating tests to account for the change.
- **[PATCH] git branch --delete-merged: handle branches with gone upstreams**: DonQuinleone proposed a hybrid of Knoble's composability idea and the original RFC's explicit opt-in, teaching `git branch -d` to accept `--upstream-gone` and `--merged=<commit>` as composable filters. Harald Nordgren reaffirmed his stance that the feature should account for "squash merge" workflows.