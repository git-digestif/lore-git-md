# Git mailing list daily digest for 2026/09/27

## The day in brief
The Git mailing list saw significant progress on the `git repo info` path-keys series, with v7 posted and all high-weight issues resolved. A bugfix for the libsecret credential helper advanced with upstream clarification and a v2 patch. The version numbering discussion expanded with proposals for a `core.defaults` meta-config variable and a consequentialist approach to default changes. A regression fix for `git stash` autostash identified a verbosity regression and a test breakage requiring further investigation.

## Notable threads

### `[PATCH v7]` series adding path-related keys to `git repo info`
K Jayatheerth posted v7 of the feature patch series adding eight new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`) to the `git repo info` command. The series is now mechanically complete and correctness-fixed, addressing all high-weight issues from v6.

Key developments in v7 include:
- **Patch 2/8 refactoring**: Split into two patches—patch 2/8 fixes `get_superproject_working_tree()` to respect the `repo` parameter and use the repository’s working directory, and patch 3/8 introduces the `path.superproject-root` keys. The fix is tested in both `t/t1900-repo-info.sh` and `t/t7400-submodule-basic.sh`.
- **`path.index` correctness**: Now relies solely on `repo_get_index_file(repo)`, which respects `GIT_INDEX_FILE` and returns the configured path even in bare repositories. The `BUG()` invocation is retained, but dead code is removed.
- **`path.cdup` edge case**: Checks `is_inside_work_tree(repo)` and, when outside the working tree, returns the absolute path to the working tree root via `repo_get_work_tree(repo)`. This aligns with `git rev-parse --show-cdup` and is tested in `t/t1900-repo-info.sh`.

The series touches `builtin/repo.c`, `Documentation/git-repo.adoc`, `t/t1900-repo-info.sh`, `submodule.c`/`submodule.h`, and `builtin/rev-parse.c`. The new keys expose filesystem locations of repository components in a scriptable format, supporting suffix-based formatting (e.g., `.absolute` and `.relative`). The series is feature-complete and ready for substantive technical review, though the two-phase refactoring plan (merge v7 as-is, then extract shared logic into `repo-info.c`) remains unresolved.

### Bugfix for `contrib/credential/libsecret`: avoid crash and silent data loss
Daniel Martí posted v2 of the bugfix for the `contrib/credential/libsecret` credential helper, addressing a crash and silent data loss issue. The v2 patch replaces the `SECRET_SEARCH_LOAD_SECRETS` flag with an explicit `secret_item_load_secret_sync()` call, ensuring errors are properly reported rather than silently discarded.

The patch removes the problematic flag from the `secret_service_search_sync()` call and explicitly loads the secret for the matched item. This change prevents assertion failures in `secret_value_get_text()` and `secret_value_unref()`, which previously caused the password to be lost even if it was retrievable. The updated commit message explains that the flag skips locked items and ignores load failures, leaving secrets as NULL, which the GNOME keyring daemon silently omits from its reply.

The patch is narrowly scoped to `contrib/credential/libsecret/git-credential-libsecret.c` and introduces no new CLI options, config keys, or on-disk format changes. The design trade-off—prioritizing simplicity and reliability over preserving the batch-fetch optimization—remains unresolved, but the upstream clarification strengthens the case for the patch.

### Git version numbering discussion: `core.defaults` and consequentialist approach
The ongoing discussion about the next Git release version number expanded with two new proposals from Phillip Wood: a consequentialist approach to default changes and a `core.defaults` meta-config variable. Junio C Hamano engaged with these ideas but raised practical concerns about their feasibility.

Phillip’s consequentialist approach challenges the project’s current deontological stance, which treats any negative impact on even a small number of users as unacceptable. He argues that for settings with limited downsides (e.g., `diff.algorithm`, `merge.conflictStyle`), the net benefit of a default change could justify the disruption. Junio acknowledged that the project already adopts this mindset for some changes (e.g., merge strategy defaults) but cautioned that "net benefit" and "potential downside" are speculative until observed in the wild.

The `core.defaults` proposal introduces a single config variable that would override defaults for a curated set of settings, allowing users to opt into "modern defaults" without forcing changes on everyone. Junio questioned its utility, suggesting that polling the value of `core.defaults` to gauge user adoption would be redundant, as the same information could be obtained by directly measuring the target configurations (e.g., how many users set `diff.algorithm` to a non-default value).

The discussion remains conceptual, with no code or implementation details, but it provides a clear path forward for revisiting default changes post-3.0. The proposals are scoped to cosmetic or workflow-related settings, avoiding high-impact changes like autostash or `git status` behavior.

### `stash: fix autostash with staged index entries`
Junio C Hamano identified a verbosity regression in the final patch of D. Ben Knoble’s bugfix series for `git stash` autostash with staged index entries. The regression occurs because the `merge_options` struct is reused for two separate merges (index and worktree), but the `.verbosity` field is unconditionally set to `0` during the index merge and never restored, causing the subsequent worktree merge to lose its configured verbosity level.

Junio proposed a fix to move the `quiet`-conditional verbosity suppression to the top of the function, ensuring the setting applies consistently to both merges. The issue is subtle: the original subprocess-based merge (`diff-tree | apply --cached`) was inherently quiet, but the new in-core merge machinery respects the user’s verbosity preferences (via `merge.verbosity` config or `GIT_MERGE_VERBOSITY` environment). The patch’s hardcoded `o.verbosity = 0` for the index merge inadvertently silences the worktree merge as well.

Additionally, Junio reported that the series breaks the `t5520` test suite when merged into his `seen` integration branch. He temporarily ejected the topic from his tree to unblock other work while the issue is investigated. The ejection is procedural and does not reflect a rejection of the series, which is otherwise technically complete and ready for integration.

## In brief
- **[PATCH] remote: clarify that `git remote set-head` affects only the local view**: Matthias Goergens posted a documentation patch clarifying that `git remote set-head` only affects the local repository’s view of the remote’s default branch, not the remote repository itself. The patch adds a short explanatory paragraph to the `git-remote` man page, addressing a common misconception.
- **[PATCH] Update `git name-rev` man page to reflect hash-algorithm-agnostic implementation**: Jyotish Kumar posted a documentation patch replacing SHA-1-specific references in the `git name-rev` man page with generic "object ID" terminology. The patch aligns the man page with the implementation, which has been hash-algorithm-agnostic since commit 1c4675dc57 (2023). Junio C Hamano and brian m. carlson acknowledged the patch as ready for merging.
- **l10n updates for Git 2.56.0**: Jiang Xin submitted a pull request with updated and expanded translations for Git’s user-facing messages. The update adds two new languages (Afrikaans and Brazilian Portuguese), refreshes eight existing translations, and modernizes the localization tooling documentation.
- **[PATCH] ci: improve leak sanitizer failure reporting in GitHub Actions**: Phillip Wood raised concerns about the `--immediate` flag hiding subsequent leaks and a GitHub Actions UI scrolling regression. Harald Nordgren acknowledged the trade-off and promised to investigate the scrolling issue but has not yet proposed a fix.
- **[PATCH] .mailmap: map kristofferhaugsbakk@fastmail.com to code@khaugsbakk.name**: Kristoffer Haugsbakk updated the `.mailmap` file to map his Fastmail and Gmail addresses to his canonical `code@khaugsbakk.name` identity, ensuring consistent display in `git check-mailmap` and related tools.
- **[PATCH] format-patch: add --[no-]range-diff-notes**: Kristoffer Haugsbakk acknowledged a redundant `test_when_finished` call in the test script and confirmed it will be fixed in the next version. The series remains in a holding state, awaiting a maintainer decision on whether to revive the feature.
- **[PATCH] interpret-trailers: improve clarity of `trailer.<key-alias>.cmd` examples**: Kristoffer Haugsbakk posted a documentation patch improving the clarity of two examples in the `git-interpret-trailers` man page. The patch replaces outdated phrasing with clearer descriptions of what the commands do, aligning with the style used elsewhere in the man page.