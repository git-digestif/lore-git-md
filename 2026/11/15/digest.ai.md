# Git mailing list daily digest for 2026/11/15

## Long threads

### `git repo info` path keys reach v9 with reviewer sign-off

*Long thread (root 2026/07/16/01-21-31): 63 replies so far from 6 participants over 123 days; 3 messages that day.*

K Jayatheerth's GSoC series exposes repository paths and metadata through the `git repo info` command, adding `path.workingtree.absolute`/`.relative`, `path.superprojectworkingtree.absolute`/`.relative`, `path.index.absolute`/`.relative`, and `path.prefix` keys, and refactoring `get_superproject_working_tree()` to operate on a `struct repository *`. Phillip Wood reviewed the v9 range-diff and said it looks good; while noting it is a shame each test still creates a new repository when most could run in the test framework's repository, he said no one else seems to mind and he would be happy for the series to be merged as-is. K Jayatheerth apologized for missing that note, explained that he is writing a follow-up series to consolidate the overlapping logic between `repo` and `rev-parse` and plans to combine and clean up the tests there, and offered to switch this series' tests to a single repository if Phillip Wood thinks it is worth doing now. Phillip Wood replied that it is easily done and, since test cleanups are planned anyway, he is happy for the tests in this series to be cleaned up by a future series.

### `credentialStore.lockTimeout` naming defended with existing conventions

*Long thread (root 2026/09/02/08-31-37): 42 replies so far from 5 participants over 75 days; 1 message that day.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries in `builtin/credential-store.c`, retaining `lockTimeoutMS` as a deprecated alias. Responding to a question about the name, Okhuomon Ajayi justified dropping the `MS` suffix by noting that other file-lock retry settings--`rerere.lockTimeout`, `reftable.lockTimeout`, `core.filesRefLockTimeout`, `core.packedRefsTimeout`, and `core.configLockTimeout`--are measured in milliseconds without the unit in the name, unlike `credentialStore.lockTimeoutMS`, while `credential-cache --timeout` is a different setting in seconds. She said she will resend with that reason in the log message.

## The day in brief
Phillip Wood gave the v9 `git repo info` path-key series his approval for merging as-is, with test consolidation deferred to a planned follow-up. Okhuomon Ajayi defended the `credentialStore.lockTimeout` name by pointing to the other millisecond-based lock-timeout settings and plans to resend with that rationale in the commit message. Andy Koppe submitted a fix in `pretty.c` for colors being lost or leaking across line breaks in custom log formats.

## Notable threads

### Preserving the `change-id` header through rebase and cherry-pick
Matt Stark's patch modifies `sequencer.c` to preserve a `change-id` header in whatever format it previously used during rebase and cherry-pick, without standardizing the format. Ewout ter Hoeven proposed stacked pull/merge requests as a concrete use case, explaining that tools such as GitHub's gh-stack and GitLab's glab stack currently guess whether a rewritten commit is the same change by relying on branch names, recorded bases, and patch IDs, with documented failures including stranded stacks after a plain branch rename. brian m. carlson objected that change-id headers cannot be trusted as a source of unmodified data because users can modify them, arguing that only a server performing the rebase can have confidence the content is unchanged, and warned that a malicious user could reuse change-ids to submit a different version under an existing review.

### Fixing lost and leaked colors at line breaks in `pretty.c`
Andy Koppe submitted a patch to `pretty.c` addressing two problems in custom log formats: colors active across line breaks are lost when output is shown without that context (notably with `--graph` and color-aware pagers such as `less`, which reset colors at line breaks), and colors still active at the end of a line leak into what follows, such as graph markers and edges. When color output is enabled, `repo_format_commit_message()` now scans the output for active color escape sequences and makes each line self-contained by inserting a reset before newlines where colors are active, replaying those colors at the start of the next non-empty line, and appending a reset at the end if colors remain active.

### Coverity overflow fixes move off GCC-only macros
Johannes Schindelin's five-patch series addresses overflows reported by Coverity across `archive-tar.c`, `http.c`, `wrapper.c`, `git-compat-util.h`, and `date.c`. In v2 updates, he changed `write_in_full()` to return 0 on success instead of the byte count and added `read_in_full_or_error()` returning 0 on success and -1 on error, while callers requesting the byte count with an oversized size now error out rather than returning a bogus negative value. He dropped the `signed_mult_overflows` and `mult_overflows` macros after Phillip Wood pointed out the type-generic macro was GCC-only, and replaced the approxidate overflow logic in `date.c` with manual overflow checks plus an `if (number > INT_MAX) return NULL` guard for the unsigned `time_t` case.

### fmt-merge-msg config doc patch retracted as redundant
Yikun Wei retracted the patch that would have included `config/fmt-merge-msg.adoc` in `Documentation/config.adoc`, explaining that `merge.branchdesc`, `merge.log`, and `merge.suppressDest` are already listed in `git-config(1)` because `config/merge.adoc` already includes `config/fmt-merge-msg.adoc` and `config.adoc` already includes `config/merge.adoc`, so the patch would duplicate those entries. Wei asked that the patch be dropped and stated they will not reroll it.

### `git apply --check` crash on patches naming both new and deleted files
Siddharth Asthana continued work on the crash where `git apply --check` hits an assertion in `check_preimage` for a patch that gives both a new and a deleted file the same name. He identified that the leak in `git_header_name()` is actually in `parse_git_diff_header()`, where `patch->def_name` is overwritten without freeing the old value, and proposed adding a `free()` before the assignment in a prep patch so the dropped test can be kept. He also suggested a cover letter with `--cover-letter --range-diff=<v2>` for v3, and noted that the error for `--- /dev/null` prints "names an old file dev/null" rather than what the user wrote, suggesting the `---` line be named before "new file mode", with the same change for the deleted case.

## In brief
- Srujith A Poojary posted v2 of the archive refactor, narrowing it to `init_tar_archiver()` and `init_archivers()` accepting an explicit `struct repository` pointer and removing `USE_THE_REPOSITORY_VARIABLE` from `archive-tar.c`, reverting the v1 changes to `archive-zip.c`, `init_zip_archiver()`, and `diagnose.c` to keep zip out of scope.
- Hardik Kumar submitted a patch replacing the `!` operator with `test_must_fail` in `t/t5564-http-proxy.sh`, `t/t6030-bisect-porcelain.sh`, `t/t7400-submodule-basic.sh`, and `t/t9001-send-email.sh`, because `test_must_fail` fails the test when a command is killed by a signal or segfaults, whereas `!` would incorrectly pass such tests.
- Yusuke Hayashi posted v2 of the `http-backend` patch dropping the `strbuf` pre-sizing from `get_info_packs()`, rewriting the commit message to explain that the one-shot CGI handler's geometric buffer growth is negligible next to CGI startup, pack discovery, and HTTP I/O; the revision makes no code changes from v1.
- Dirk Gouders resent `[PATCH v3] doc/gitcli: minor fixes` with proper Cc and a range-diff against v1, fixing plural/singular usage and imperative form in `Documentation/gitcli.adoc` and asking that v2 be disregarded.
