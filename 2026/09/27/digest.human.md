# Git mailing list daily digest for 2026/09/27

## The day in brief
The Git mailing list saw significant progress on a long-running feature series adding path-related keys to `git repo info`, with the v7 iteration now mechanically complete and ready for substantive review. A critical bugfix for the `contrib/credential/libsecret` helper advanced with upstream clarification, while a regression in `git stash` autostash handling was identified and a fix proposed. The version numbering discussion for Git 3.0 expanded with new proposals for handling default changes, and several documentation patches improved clarity across man pages.

## Notable threads

### Path-related keys for `git repo info` reach v7
The v7 iteration of K Jayatheerth’s series adding eight new path-related keys to `git repo info` was posted today, marking the series as mechanically complete and correctness-fixed. All high-weight issues from v6 have been resolved, including a critical bug in `get_superproject_working_tree()` that previously ignored the `repo` parameter, and an edge-case discrepancy in `path.cdup` that now matches `git rev-parse --show-cdup` behavior exactly.

The series introduces keys like `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`, exposing filesystem locations of repository components in a scriptable format. Each key supports suffix-based formatting (e.g., `.absolute` and `.relative`), with empty strings returned for inapplicable contexts like bare repositories. The implementation touches `builtin/repo.c`, `submodule.c`, and adds test coverage in both `t/t1900-repo-info.sh` and `t/t7400-submodule-basic.sh`.

The architectural concern about shared logic between `git repo info` and `git rev-parse` remains deferred to a follow-up refactoring series, as proposed by the author. The series is now ready for in-depth technical review, with no known open issues beyond this deferred work.

### Libsecret credential helper bugfix advances with upstream clarification
Daniel Martí posted v2 of a bugfix for the `contrib/credential/libsecret` helper, addressing a crash and silent data loss issue caused by the `SECRET_SEARCH_LOAD_SECRETS` flag. The v2 patch replaces the flag with an explicit `secret_item_load_secret_sync()` call, ensuring errors like locked or concurrently deleted items are properly reported rather than silently discarded.

Martí confirmed that the flag’s behavior—silently skipping inaccessible items—is intentional upstream, as the GNOME keyring daemon omits such items during batch fetches. This means Git must handle NULL secrets regardless of libsecret’s version, and the patch aligns with libsecret’s own `secret-tool` utility. The v2 patch also improves the commit message to address Junio C Hamano’s earlier feedback about clarity, though the technical approach remains unchanged.

The unresolved design question is whether to preserve the batch-fetch optimization (Junio’s preference) or prioritize simplicity and reliability (Martí’s approach). The upstream clarification strengthens the case for the latter, as it confirms the flag’s behavior is intentional and that Git must handle NULL secrets either way.

### Verbosity regression identified in `git stash` autostash fix
Junio C Hamano identified a verbosity regression in the final patch of D. Ben Knoble’s five-part bugfix series for `git stash` autostash with staged index entries. The issue arises because the `merge_options` struct is reused for two separate merges (index and worktree), but the `.verbosity` field is unconditionally set to `0` during the index merge and never restored. This causes the subsequent worktree merge to lose its configured verbosity level, potentially breaking scripts or workflows relying on verbose output.

Junio proposed a minimal fix to move the `quiet`-conditional verbosity suppression to the top of the function, ensuring the setting applies consistently to both merges. The series as a whole (patches 2–5) was previously ready for integration but has now been ejected from Junio’s `seen` branch due to a test breakage in `t5520` when merged into `seen`. The ejection is procedural to unblock other work and does not reflect a rejection of the series.

### Version numbering discussion expands with new proposals
The ongoing discussion about the next Git release’s version number saw new proposals from Phillip Wood and engagement from Junio C Hamano. Phillip introduced two ideas in response to Junio’s rejection of user-facing breaking changes in Git 3.0: (1) a consequentialist approach to default changes for low-impact settings, and (2) a `core.defaults` meta-config variable that would allow users to opt into "modern defaults" without disruptive breaking changes.

Junio questioned the practicality of `core.defaults`, suggesting that polling the value of target configurations (e.g., `diff.algorithm`) might be more effective than an indirect proxy. He also noted that "net benefit" and "potential downside" of default changes are speculative until observed in the wild. The discussion remains open, with no consensus on how to handle default changes in future releases.

## In brief
- **`git remote set-head` documentation**: Matthias Goergens posted a patch clarifying that `git remote set-head` affects only the local view of the remote’s default branch, not the remote repository itself.
- **`.mailmap` update**: Kristoffer Haugsbakk updated the `.mailmap` file to map his Fastmail and Gmail addresses to his canonical `code@khaugsbakk.name` identity.
- **`git name-rev` man page**: Jyotish Kumar posted a patch replacing SHA-1-specific references in the `git name-rev` man page with generic "object ID" terminology, aligning with the hash-algorithm-agnostic implementation.
- **Localization updates**: Jiang Xin submitted l10n updates for Git 2.56.0, adding Afrikaans and Brazilian Portuguese translations and refreshing eight existing languages.
- **`git-interpret-trailers` documentation**: Kristoffer Haugsbakk posted a patch improving clarity of `trailer.<key-alias>.cmd` examples in the `git-interpret-trailers` man page.
- **`format-patch` range-diff notes**: Kristoffer Haugsbakk acknowledged a redundant `test_when_finished` call in the v2 series adding `--[no-]range-diff-notes` to `git format-patch`, confirming it will be fixed in the next version.
- **CI leak sanitizer reporting**: Phillip Wood raised concerns about the `--immediate` flag hiding subsequent leaks and a GitHub Actions UI scrolling regression in Harald Nordgren’s CI improvement patch. Harald agreed to expand the commit message but has not yet addressed the substantive concerns.