# Git mailing list daily digest for 2026/09/24

## Long threads

### http.sslVerifyStatus for stapled OCSP responses

*Long thread (root 2026/08/11/17-02-00): 29 replies so far from 5 participants over 45 days; 3 messages that day.*

This topic adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS`, because git never sets that option and therefore ignores stapled OCSP responses, including ones stating a certificate is revoked; it defaults to false so connections to servers without OCSP are not blocked. Earlier, the combined v7 was reviewed, merged to `next`, and then reverted after SZEDER Gábor reported a t5585 failure on his system. Today, SZEDER Gábor concluded that the `SSL_VERIFYSTATUS` prereq only verifies libcurl supports `CURLOPT_SSL_VERIFYSTATUS` and says nothing about the server, so it should check for the absence of "could not enable OCSP status verification" rather than "http.sslVerifyStatus is set", and renaming the prereq `CURL_SSL_VERIFYSTATUS` might have made this clearer sooner. Patrick Steinhardt asked whether SZEDER might be running a curl backend without proper OCSP support, while noting the test suite should skip in that case. Junio C Hamano affirmed both of SZEDER's suggestions as understandable.

### reference-transaction hooks reporting old values

*Long thread (root 2026/09/19/13-34-15): 25 replies so far from 5 participants over 6 days; 9 messages that day.*

The reference-transaction hook currently reports an all-zero old object ID for unconditional updates, so batched branch, tag, and remote-ref deletions report zero for both old and new values. Today's discussion moved the fix into the common transaction layer: Karthik Nayak reviewed v5 and objected to replacing the `null_oid` BUG() in `ref_transaction_delete()`; Patrick Steinhardt argued callers needing expected old OIDs should drive their own transactions and instead suggested resolving the old value in the common layer for the hook; Junio C Hamano countered that a pointer to `null_oid` is a valid "N/A" representation. Maciej Ciemborowicz agreed to drop the caller-specific API changes and posted v6, a single transaction-layer patch that resolves old values before the "preparing" hook and refreshes them after the backend locks the refs, only when a hook exists. The full test suite passes (1060 files, 34660 tests), and the focused reference-transaction tests pass with SHA-1 and SHA-256 using both files and reftable backends.

### GitLab CI Windows cargo invocation fix

*Long thread (root 2026/09/19/12-12-09): 12 replies so far from 3 participants over 6 days; 8 messages that day.*

This four-patch series fixes the GitLab CI cargo invocation in Windows jobs, broken since Rust was enabled there. Today, Patrick Steinhardt asked what "GNU Rust" means, whether the `PATH` assignment needs quoting, and whether patch 4 leaves two GCC versions available. Johannes Schindelin clarified that "GNU Rust" means the Rust toolchain targeting GCC, declined hoisting `install-sdk.ps1` into `install-dependencies.ps1`, explained the `Gcc` feature only ensures Rust can produce libraries GCC can link to rather than installing a second GCC, and posted the v2 reroll rewording the commit messages in patches 1 and 4.

## The day in brief
Junio C Hamano laid out a proposed Git v3.0 timeline: 2.98 at the end of this year, 2.99 in March 2027, 2.99.1 in April 2027 with only bugfixes, and 3.0 identical except with breaking changes enabled. Maciej Ciemborowicz posted v6 of the reference-transaction hook fix as a single transaction-layer change after reviewers steered him away from extending `refs_delete_refs()`. Patrick Steinhardt posted two series: one making the repository parameter of `create_repository()` a pure out-parameter, and one with Meson build improvements including a fix for hanging MSVC CI jobs.

## Notable threads

### Git v3.0 release timeline
Junio C Hamano proposed the v3.0 timeline described above and cautioned it is not set in stone: 2.99.1 could slip or be followed by 2.99.2, and anything beyond this year is "play it by ear". Johannes Schindelin thanked him, saying the timeline helps plan the Git for Windows roadmap including Rust-based builds and Git Credential Manager v3.0. Ramsay Jones reported that building master on Cygwin with the experimental Rust package succeeded, with test failures identical to v2.56.0-rc2.

### create_repository() stateless repository parameter
Patrick Steinhardt posted a seven-patch series making the repository passed to `create_repository()` a pure out-parameter by calling `repo_clear()` first, eliminating the last in/out state, `core.sharedRepository`. The series moves `core.sharedRepository` handling from `builtin/init-db.c` into `setup.c`, adapts `repo_clear()` to zero the whole structure with `memset()`, switches `git-init` and `git-clone` to a const variant of leading-directory creation that ignores the shared-repository setting, and adds tests to `t/t1301-shared-repo.sh`.

### Meson build improvements and hanging MSVC jobs
Patrick Steinhardt posted a seven-patch Meson series: linking the HTTP sources into a static library, enabling precompiled headers for the test tool (~19% clean-build speedup) and unit tests, fixing outdated completion helpers, updating subproject wrappers, and fixing hanging GitLab MSVC jobs by unsetting the system `credential.helper` after installing PortableGit so Runner 19.x's cleanup-stage `git credential reject` cannot hang in Git Credential Manager.

### gitmergeconflicts(7) documentation page

Julia Evans posted a seven-patch series introducing a new `gitmergeconflicts(7)` guide explaining merge-conflict resolution with examples, linked from the `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull` man pages. Junio C Hamano suggested moving the `.gitattributes` conflict-marker-size change into the first patch and reported that in `seen`, `make check-docs` fails with "no link: gitmergeconflicts"; Jeff King explained the failure appears only there because the command-list lint is not yet in master and suggested listing the guide in `command-list.txt`.

### reflog default expiry periods fix
The default reflog expiry periods were swapped when moved to `REFLOG_EXPIRE_OPTIONS_INIT()`, expiring reachable entries after 30 days instead of 90 and unreachable ones after 90 instead of 30. Pushkar Singh posted v3 splitting the regression test into four independent cases; Patrick Steinhardt asked whether to fast-track it for Git 2.56, Jeff King noted the breakage dates to v2.50.0 but called the fix low-risk, and Junio C Hamano judged the v3 tests much more nicely organized.

## In brief
- Phillip Wood spotted a memory leak in the `builtin/stash` in-core index merge (missing `merge_finalize`), and Junio C Hamano questioned the tree order passed to `merge_incore_nonrecursive()`.
- Patrick Steinhardt retracted an earlier objection and said the object-file packfile-flush v2 series looks good to him.
- **Knoble and Hamano endorse `fetch.<remote>.refmap`**: D. Ben Knoble agreed `fetch.<remote>.refmap` is the right approach for `git fetch second main` to create remote-tracking branches, which Junio C Hamano confirmed would work as wanted.

- Johannes Schindelin posted v2 of the Debian 12 HTTP/2 workaround using a `HAVE_CURL_HTTP2_BUG` prereq that skips two half-auth tests for libcurl 7.88.0 through pre-8.3.0; Jeff King said it looks good.
- Kaartic Sivaraam posted an RFC series making setup errors state why a directory is not a valid repository; Junio C Hamano pointed out style nits and dead `strbuf_release()` after `die()`.
- Jeff King reported that per-file `EXTRA_CPPFLAGS` in `config.mak` makes `make` warn the precompiled header is unused.
- Tamir Duberstein benchmarked `cmp` versus `diff` on huge output (0.506 s and 1,264 KiB versus 5.276 s and 4,199,924 KiB) while responding to Patrick Steinhardt's questions about the CI large-fixture patch.
- Julia Evans, Jeff King, and Junio C Hamano worked through AsciiDoc cross-reference rendering; Junio marked the topic expecting a reroll.
- Harald Nordgren argued Git need not signal 3.0 in advance; Junio C Hamano rejected that, explaining the version jump warns downstream projects and distributions.
- Nathan Froyd submitted a patch setting `exact_oid` in `builtin/fetch-pack.c` so `git fetch-pack $OID` sends `want` rather than `want-ref`.
- Mark C. Chu-Carroll added `--no-branches`, `--no-tags`, `--no-remotes`, `--no-notes`, and `--no-stashes` filtering flags to `git repo structure`.
- Junio C Hamano proposed an alternative `git-contacts` stdin implementation treating `-` as a file argument read from `STDIN`.
