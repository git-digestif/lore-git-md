# Git mailing list daily digest for 2026/11/15

## Long threads

### `git repo info` path keys series reaches v9 review

*Long thread: 63 replies so far from 6 participants over 123 days; 3 messages that day.*

K Jayatheerth's GSoC series extends the `git repo info` command with path keys such as `path.workingtree.absolute`/`path.workingtree.relative`, `path.superprojectworkingtree.*`, `path.index.*`, and `path.prefix`, and refactors `get_superproject_working_tree()` to operate on a `struct repository *`. Phillip Wood reviewed the v9 range-diff and said it looks good; while noting it is a shame that each test still creates a new repository when most could run in the test framework's repository, he said no one else seems to mind and he would be happy for the series to be merged as-is. K Jayatheerth apologized for missing that note, explained that he is writing a follow-up series to consolidate the overlapping logic between `repo` and `rev-parse` and plans to combine and clean up the tests there, and offered to switch this series' tests to a single repository if Phillip Wood thinks it is worth doing now. Phillip Wood replied that it is easily done and, since test cleanups are planned anyway, he is happy for the tests in this series to be cleaned up by a future series.

### `credentialStore.lockTimeout` naming defended

*Long thread: 42 replies so far from 5 participants over 75 days; 1 message that day.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` in `builtin/credential-store.c` as the preferred setting for credential lock retries, retaining `lockTimeoutMS` as a deprecated alias. Responding to a question about the unit-less name, she justified it by noting that other file-lock retry settings--`rerere.lockTimeout`, `reftable.lockTimeout`, `core.filesRefLockTimeout`, `core.packedRefsTimeout`, and `core.configLockTimeout`--are measured in milliseconds without the unit in the name, unlike `credentialStore.lockTimeoutMS`, while `credential-cache --timeout` is a different setting in seconds. She said she will resend with that reason in the commit message.

## The day in brief

Phillip Wood reviewed the v9 `git repo info` path-keys series and said he would be happy for it to be merged as-is, with test consolidation deferred to a planned follow-up series. Okhuomon Ajayi defended the `lockTimeout` name for the credential-store lock-retry setting by pointing to the other millisecond-based lock timeouts and plans to resend with the rationale in the log message. Andy Koppe submitted a patch to `pretty.c` fixing colors that are lost or leaked at line breaks in custom log formats.

## Notable threads

### Preserving the `change-id` header through rebase and cherry-pick

Matt Stark's patch modifies `sequencer.c` so that rebase and cherry-pick preserve a `change-id` header in whatever format it previously used, without standardizing the format. Ewout ter Hoeven proposed stacked pull/merge requests as a concrete use case, explaining that tools such as GitHub's gh-stack and GitLab's glab stack currently guess whether a rewritten commit is the same change by relying on branch names, recorded bases, and patch IDs, with documented failures including stranded stacks after a plain branch rename. brian m. carlson objected that change-id headers cannot be trusted as a source of unmodified data because users can modify them, arguing that only a server performing the rebase can have confidence the content is unchanged, and warned that a malicious user could reuse change-ids to submit a different version under an existing review.

### fmt-merge-msg config doc patch retracted

Yikun Wei's documentation patch would have included `config/fmt-merge-msg.adoc` in `Documentation/config.adoc` so that `merge.branchdesc`, `merge.log`, and `merge.suppressDest` appear in `git-config(1)`. Wei retracted the patch, explaining that those entries are already listed in `git-config(1)` because `config/merge.adoc` already includes `config/fmt-merge-msg.adoc` and `config.adoc` already includes `config/merge.adoc`, so the patch would duplicate them; Wei asked that the patch be dropped and will not reroll it.

### Fix for lost and leaked colors at line breaks in `pretty.c`

Andy Koppe submitted a patch to `pretty.c` fixing lost and leaked colors at line breaks in custom log formats: when color output is enabled, `repo_format_commit_message()` now scans the output for active color escape sequences and makes each line self-contained by inserting a reset before newlines where colors are active, replaying those colors at the start of the next non-empty line, and appending a reset at the end if colors remain active. This prevents colors from being lost when `--graph` or color-aware pagers reset at line breaks, and from leaking into graph markers, edges, or unrelated lines when grepping colored output.

## In brief

- Johannes Schindelin updated his Coverity overflow series: `write_in_full()` now returns 0 on success, the GCC-only `mult_overflows` macros were dropped, and `date.c` uses manual overflow checks in the approxidate relative-time multiplication.
- Siddharth Asthana traced the `git apply --check` crash leak to `parse_git_diff_header()` overwriting `patch->def_name` without freeing the old value, and suggested wording fixes for the "--- /dev/null" error message.
- Srujith A Poojary posted v2 of the archive refactor, narrowed to passing an explicit `struct repository` to `init_tar_archiver()` and `init_archivers()` and removing `USE_THE_REPOSITORY_VARIABLE` from `archive-tar.c`.
- Hardik Kumar replaced the `!` operator with `test_must_fail` in `t/t5564-http-proxy.sh`, `t/t6030-bisect-porcelain.sh`, `t/t7400-submodule-basic.sh`, and `t/t9001-send-email.sh` so that signal-killed or segfaulting commands fail the test.
- Yusuke Hayashi posted v2 of the `http-backend` patch dropping the strbuf pre-sizing in `get_info_packs()`, with a rewritten commit message and no code changes from v1.
- Dirk Gouders resent v3 of the `doc/gitcli` minor fixes with the proper Cc and a range-diff against v1.
