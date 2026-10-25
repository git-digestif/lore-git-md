# Git mailing list daily digest for 2026/10/24

## The day in brief
The Git mailing list saw significant activity around the `remote.<name>.refmap` fetch series (now ready for `next`), a philosophical debate about Git's defaults for triangular workflows, and multiple refactoring efforts in the libification project. Key developments include a fix for the reflog `--operations` feature, progress on the object storage extension RFC, and several bug reports with proposed fixes.

## Notable threads

### `remote.<name>.refmap` fetch series ready for `next`
**What changed**: Harald Nordgren's v10 series introducing `remote.<name>.refmap` for matching-inspired fetch mode is now ready for merging to `next` after addressing all review feedback, including a bugfix for the fallback logic in `remote_find_tracking()`.

**Problem/goal**: Users of shallow repositories experience slow fetches when the remote has many branches, because a refspec-less fetch negotiates history for all remote branches. The series implements a "matching-inspired" fetch mode that dynamically fetches only the branches tracked by local branches at the remote.

**Subsystem**: Remote configuration and fetch logic (`remote.c`, `builtin/fetch.c`).

**Impact**: The series reduces unnecessary history negotiation in shallow repositories by inferring which branches to fetch based on local tracking branches. The new `--limited-fetch` flag for `git remote add` and shallow-repository defaults make this behavior more accessible.

**Today's developments**:
- [2026/10/24/15-08-15] D. Ben Knoble raised a technical question about the fallback logic in `remote_find_tracking()`: whether the code should fall back to `remote->refmap` only when `remote->fetch` is empty, or also when `refspec_find_match()` fails to find a match in `remote->fetch`. The concern was that this fallback might cause the function to return a refspec from `remote->refmap` even when the user intended to query only the explicitly configured `remote->fetch` refspecs.
- [2026/10/24/17-18-57] Harald Nordgren fixed the fallback logic to ensure it only falls back to `remote->refmap` when `remote->fetch` is either empty *or* contains no matching refspec for the given query. The change replaces the original two-branch `if/else` with a single condition that checks both the presence of `remote->fetch` refspecs and the success of `refspec_find_match()`.

**Status**: The series is technically complete and ready for `next`. The procedural blocker (a second reviewer's Ack) has been satisfied by D. Ben Knoble's implicit endorsement.

---

### Philosophical debate: Git's defaults for triangular workflows
**What changed**: A heated discussion about whether Git should adapt its defaults to accommodate users shaped by GitHub's fork-and-pull model, or resist such changes to avoid reinforcing broken mental models of Git's branching concepts.

**Problem/goal**: Harald Nordgren proposed making `git status` compare against both `@{upstream}` and `@{push}` by default to better support triangular workflows (e.g., GitHub PRs). Junio C Hamano pushed back, arguing that `@{push}` divergence is less critical to monitor by default because it reflects the user's own work, not external changes.

**Subsystem**: Porcelain (`git status`), remote tracking.

**Impact**: The debate touches on Git's cultural identity and how it balances backward compatibility with usability improvements for modern workflows. The outcome could influence future defaults and the project's stance on accommodating GitHub-style workflows.

**Today's developments**:
- [2026/10/24/13-49-26] D. Ben Knoble supported the patch with a pragmatic, user-centric argument, describing his own workflow where he frequently checks divergence between local and `@{push}` branches to answer questions like *"Have I rebased lately and need to force-push?"*. His personal tooling (`git sbup`, `git rdup`, `git div`) demonstrates that this need is real and widespread among users who work in triangular setups.
- [2026/10/24/16-43-00] Junio refined his critique, distinguishing between the *nature* of changes in `@{upstream}` versus `@{push}`. He argued that while `@{upstream}` may introduce "surprises" (unexpected commits from others), `@{push}` reflects the user's own work, making its divergence less critical to monitor by default. He also proposed that external tools (e.g., GitHub) should promote the configuration alongside the `-u` flag, preserving internal consistency without disrupting existing workflows.

**Status**: The debate remains unresolved, with the discussion now focused on whether `@{push}` divergence is inherently less important than `@{upstream}` divergence. Junio's latest reply narrows the focus to this technical distinction, but the philosophical and cultural dimensions persist.

---

### Reflog `--operations` and `--historical-branch` redesign
**What changed**: Phillip Wood's substantive review of Harald Nordgren's `git reflog` feature series identified major technical concerns, leading to a proposed redesign that replaces the dedicated flags with format specifiers (e.g., `%gb`).

**Problem/goal**: The series aimed to improve reflog usability by adding `--operations` (collapsing intermediate reflog entries from multi-step operations) and `--historical-branch` (showing which branch HEAD was on when each entry was recorded). Phillip Wood's review challenged the viability of `--operations` and identified a critical correctness gap in `--historical-branch`.

**Subsystem**: Reflog (`reflog-walk.c`, `revision.c`).

**Impact**: The proposed redesign would make the features more composable and align with Git's existing decoration system, but it requires a ground-up reimplementation. The discussion highlights the tension between usability improvements and architectural purity.

**Today's developments**:
- [2026/10/24/13-51-13] Phillip Wood raised two major technical concerns:
  1. `--operations` oversells its capabilities: not all commands update the reflog (e.g., `git cherry-pick A B C` creates multiple indistinguishable entries), and some operations leave no trace at all.
  2. Proposed replacing `--historical-branch` with a format specifier (e.g., `%gb`) to align with Git's existing decoration system, and identified a critical correctness gap in `--historical-branch`’s handling of `git rebase <upstream> <branch>`, where the branch name in the "(finish)" message must be compared to the pre-rebase checkout (requiring backward reflog traversal). Linked to a patch demonstrating the complexity of this edge case.
- [2026/10/24/15-22-28] D. Ben Knoble endorsed Phillip’s critique of both features, calling the heuristic-based approach "always going to be a bit heuristic" and noting that a true operations log would require a separate subsystem. Supported Phillip’s proposal to implement `--historical-branch` as a format specifier (e.g., `%gb`) for composability.
- [2026/10/24/16-40-05] Junio endorsed the *direction* of the series—making reflog output easier to understand—but proposed a broader refactoring of multi-step commands (e.g., `git am`, `git cherry-pick`) to work on detached HEAD by default, which would naturally produce cleaner reflog entries without needing a heuristic-based filter. Also suggested changing `git reflog`’s default behavior to operate on the current branch rather than HEAD.
- [2026/10/24/19-12-45] Harald Nordgren tentatively accepted Phillip’s proposal to replace `--historical-branch` with a format specifier (`%gb`), calling it a "radical change" that would also drop `--operations` from the series. Deferred to Phillip’s linked patch, which demonstrates the complexity of the rebase edge case and would require backward reflog traversal to determine the correct branch name.

**Status**: The series now hinges on whether Phillip’s format-specifier approach can be implemented robustly. If so, the series would need a ground-up redesign, shifting from dedicated flags to composable formatting options.

---

### Object storage extension RFC: architectural rationale questioned
**What changed**: SZEDER Gábor's review of the object storage extension RFC raised a fundamental design question: whether the object storage backend should be user-configurable at all.

**Problem/goal**: The series introduces `extensions.objectStorage` as a repository-level configuration key to enable alternative object storage backends (e.g., cloud storage, databases). The goal is to make Git's object database pluggable, following the pattern of the ref storage extension.

**Subsystem**: Object database (`odb/`).

**Impact**: The outcome of this discussion will determine whether the object storage backend is exposed as a public API (justifying end-user control) or treated as a plumbing mechanism (where configurability may be over-engineering). The decision could influence the design of future pluggable subsystems.

**Today's developments**:
- [2026/10/24/14-50-20] SZEDER Gábor challenged the **architectural rationale** behind user-configurable object storage backends. The review does not engage with implementation details but questions whether `extensions.objectStorage` should exist as a public API. The reviewer’s concern is **not about how the feature works** but about **whether it should exist at all**, given that the `files` backend’s payload only relocates `.git/objects`. The question implies skepticism about the parallel with `extensions.refStorage` and suggests that backend selection might be better handled internally (e.g., via `git clone`/`git init` options).

**Status**: The thread’s focus has shifted from mechanical correctness to **architectural scope**. The series’ cover letter and prior patches frame the feature as enabling pluggable backends, but the reviewer’s question implies that this goal may not inherently require user-facing configurability. The author’s response will need to clarify whether the extension is intended as a **public API** or a **plumbing mechanism**.

---

### Libification: line-ending configuration refactoring
**What changed**: Aniket Gupta posted a four-patch series moving all line-ending configuration variables (`core.autocrlf`, `core.eol`, `core.safecrlf`) from global variables into `struct repo_config_values` as part of the Git libification effort.

**Problem/goal**: Eliminate global state for line-ending configuration to advance Git's libification, making the codebase more modular and reusable. The change follows the established pattern of prior conversions in the libification effort.

**Subsystem**: Line-ending conversion (`convert.c`, `environment.c`).

**Impact**: The series is a mechanical but necessary step in the broader libification effort, which aims to make Git's codebase more library-like by removing global variables and making functions take explicit parameters. The changes are confined to internal APIs and do not affect user-visible behavior.

**Today's developments**:
- [2026/10/24/22-00-08] Aniket Gupta posted the complete four-patch series:
  1. Patch 1/4 passes a `struct repository *` parameter to working tree conversion functions so they can access the repository-specific configuration values. The parameter is marked `UNUSED` for now, as the actual migration of the configuration variables happens in later patches.
  2. Patch 2/4 moves the `core.autocrlf` setting from the global variable `auto_crlf` into `struct repo_config_values`.
  3. Patch 3/4 moves the `core.eol` setting from the global variable `core_eol` into `struct repo_config_values`.
  4. Patch 4/4 moves the last remaining variable, `global_conv_flags_eol` (which holds `core.safecrlf`), into `struct repo_config_values` and introduces the getter `repo_global_conv_flags_eol()`.

**Status**: The series is now fully posted and ready for review. The main point of discussion is likely to be the design choice between passing `repo` to functions (the current approach) and storing resolved values in `struct conv_attrs` (the alternative).

---

## In brief
- **[2026/10/24/11-18-56]** Tuomas Ahola surfaced a discrepancy between Peff’s original suggestion at the Contributor Summit (50 merged patches as the threshold for voting eligibility) and the much lower thresholds being discussed in the patch review (e.g., Junio’s five commits). The email refocuses the discussion on the original intent: Peff’s higher threshold was meant to ensure voters were "active developers with a track record," while the current proposals risk lowering the bar too far.
- **[2026/10/24/18-30-56]** Torsten Bögershausen submitted a platform-compatibility bugfix for a test in the recently merged `ps/pack-check-bigfile-threshold` topic. The patch addresses a macOS-specific failure in `t/t1450-fsck.sh` where the BSD `mv` utility prompts for confirmation before overwriting a read-only file, causing the test to hang or fail. The fix replaces `mv obj.tmp "$obj"` with `mv -f obj.tmp "$obj"` to suppress the prompt.
- **[2026/10/24/06-10-17]** Tanga Joris confirmed a long-standing bug in `git apply --check` that causes a crash when processing a patch that creates a new file with the header ordering `--- /dev/null` before `new file mode`. The bug is reproducible on both current `master` and Git 2.43, indicating it is not a recent regression.
- **[2026/10/24/09-22-21]** Shashank Srikumar posted v3 of the `prune-packed` refactoring patch, addressing all prior feedback: removed the `[Outreachy Patch]` prefix from the subject line, omitted the explicit listing of the three builtin files from the commit message, and clarified that the `repo` pointer *can* be `NULL` when the command is invoked with `-h`.
- **[2026/10/24/16-47-26]** Junio C Hamano acknowledged Matt Hunter’s approval of the `remote.<name>.followRemoteHEAD` lazy-validation patch and noted a **textual conflict** with Harald Nordgren’s in-flight `hn/fetch-refmap` topic, which is currently keeping this patch out of the `seen` integration branch. Suggested waiting for `hn/fetch-refmap` to land in `next` (or `master`) before rebasing this patch.
- **[2026/10/24/01-38-50]** Laveesh Bansal submitted a bugfix patch for `git reflog expire --dry-run --verbose`, fixing a UI inconsistency where the output says "prune" instead of "would prune" for named refs (while `--all` correctly uses "would prune"). The patch ensures the `dry_run` flag is propagated into the callback struct for named refs, matching the existing behavior for `--all`.
- **[2026/10/24/05-24-49]** Tanga Joris submitted a small but user-visible fix to Git’s command-line completion. The patch removes the `NO_PARSEOPT` flag from four built-in commands (`patch-id`, `diff-pairs`, `fast-import`, and `var`) that have already been converted to use Git’s `parse-options` API but were still incorrectly marked as hand-parsing their options.
- **[2026/10/24/06-47-53]** jyotish kumar proposed a test modernization for `t1006-cat-file.sh`, addressing a long-standing `FIXME` where the `%(rest)` format atom in `git cat-file --batch-check` fails when the object name contains whitespace (e.g., `HEAD:path/to/a/file with spaces`). The test currently marks this case as an expected failure (`test_expect_failure`). The author proposes using the resolved object ID (`$oid`) instead of the original object name, which may contain whitespace, to make the test pass.
- **[2026/10/24/15-29-31]** Aniket Gupta posted v2 of the i18n bugfix patch for the reflog error message pluralization. The patch updates two tests (`t1400-update-ref.sh` and `t1506-rev-parse-diagnosis.sh`) to account for the change: the new tests in `t1400` verify the exact error message for both singular and plural cases, while `t1506` relaxes its grep pattern to match either "entry" or "entries" (using "entr" as a common substring).
- **[2026/10/24/17-59-55]** Daniel Zoghalchali reported a regression in Git 2.56.0 (commit d43b9acb05) where a fetch combining `--filter=tree:0` with `--unshallow` (or `--deepen`) aborts with a BUG() in `should_include_obj()` when the fetched ref has moved past the local shallow tip. The issue stems from the environment-based shallow file propagation introduced in d43b9acb05: the `pack-objects` grandchild process inherits an empty `GIT_SHALLOW_FILE`, causing it to walk past the shallow boundary into promisor objects whose trees were never fetched, triggering the assertion.
- **[2026/10/24/18-50-42]** Sami Jawhar submitted a bugfix for the oss-fuzz integration, addressing a regression introduced in commit f1141b4391 (2025-08-15) where `parse_commit_graph()` was refactored to take a repository parameter and now calls `prepare_repo_settings()`. The fuzzer’s repository lacks a `gitdir`, causing `prepare_repo_settings()` to trigger a BUG assertion before the fuzzer can test optional chunks like generation data or Bloom filters. The fix sets a placeholder `.git` gitdir (freed by `repo_clear()`) to satisfy the check in `prepare_repo_settings()`.
- **[2026/10/24/22-13-41]** Okhuomon Ajayi submitted a refactoring patch that removes reliance on the `the_repository` global variable in `diffcore-rename.c`. The patch updates `diffcore_rename_extended()` to use the repository pointer already available in the `diff_options` struct instead of implicitly referencing `the_repository`. Specifically, it passes `options->repo->hash_algo` to `is_empty_blob_oid()` and `options->repo` to `start_delayed_progress()`. The `USE_THE_REPOSITORY_VARIABLE` macro is removed as it is no longer needed.