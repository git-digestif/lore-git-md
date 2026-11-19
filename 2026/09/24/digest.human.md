# Git mailing list daily digest for 2026/09/24

## Long threads

### http.sslVerifyStatus OCSP stapling verification

*Long thread: 29 replies so far from 5 participants over 45 days; 3 messages that day.*

Git's HTTP transport never sets `CURLOPT_SSL_VERIFYSTATUS`, so libcurl never requests the OCSP "Certificate Status Request" extension and stapled responses--including ones stating a certificate is revoked--are ignored. The series adds an `http.sslVerifyStatus` boolean mapping to that option, defaulting to false because enabling it blocks connections to servers that send no stapled response. Earlier, the topic was reverted out of `next` after SZEDER Gábor reported a t5585 failure. Today SZEDER Gábor concluded the `SSL_VERIFYSTATUS` prereq only verifies libcurl supports `CURLOPT_SSL_VERIFYSTATUS` and says nothing about server capabilities, so it should check for the absence of "could not enable OCSP status verification" rather than "http.sslVerifyStatus is set", and renaming it `CURL_SSL_VERIFYSTATUS` might have helped. Patrick Steinhardt asked whether SZEDER might run a curl backend without proper OCSP support, noting the suite should skip in that case. Junio C Hamano affirmed both suggestions as understandable.

### reference-transaction hooks reporting old values

*Long thread: 25 replies so far from 5 participants over 6 days; 9 messages that day.*

The reference-transaction hook reports an all-zero old object ID for unconditional updates, so batched branch, tag, and remote-ref deletions report zero for both old and new values. After earlier iterations extended `refs_delete_refs()` and its callers, today Maciej Ciemborowicz posted [PATCH v6] as a single transaction-layer change: the common hook layer resolves the old value before the "preparing" hook and refreshes it after the backend locks the references, only when a hook exists, keeping the hook value separate from caller-supplied `old_oid` so it cannot constrain the update. Patrick Steinhardt had argued for exactly this common-layer approach earlier in the day, and Maciej agreed the batch-deletion API changes were the wrong layer, leaving `refs_delete_refs()`, `ref_transaction_delete()`, and all callers unchanged.

### GitLab CI Windows cargo invocation fix

*Long thread: 12 replies so far from 3 participants over 6 days; 8 messages that day.*

A four-patch series fixes the GitLab CI cargo invocation in the Windows jobs, broken since Rust was enabled there. Today Johannes Schindelin posted [PATCH v2], rewording commit messages in patches 1 and 4 (patch 1 now says "Rust toolchain that targets GCC"; patch 4 corrects "GNU MSI" to "Rust MSI"), with patches 2 and 3 unchanged. He answered Patrick Steinhardt's questions: "GNU Rust" meant the GCC-targeting toolchain, `install-dependencies.ps1` is not yet used for the MinGW build, the unquoted PATH assignment needs no quoting, and patch 4 installs no second GCC--the `Gcc` feature only ensures the installed Rust can produce libraries GCC can link to.

## The day in brief
Junio C Hamano laid out a proposed Git v3.0 timeline (2.98 at year's end, 2.99 in March 2027, 2.99.1 in April 2027, and 3.0 differing only in enabled breaking changes), while cautioning it is not set in stone. Maciej Ciemborowicz posted v6 of the reference-transaction hook fix as a single transaction-layer change, and Johannes Schindelin rerolled the GitLab CI Windows cargo fix as v2.

## Notable threads

### Git v3.0 timeline proposal
Junio C Hamano proposed a v3.0 timeline: 2.98 ending this year, 2.99 in March 2027 as a signal that something big is coming, 2.99.1 in April 2027 carrying only accumulated bugfixes, and 3.0 identical to 2.99.1 except with breaking changes enabled, keeping the `WITH_BREAKING_CHANGES` Makefile knob. He later cautioned that anything beyond this year is "play it by ear" territory. Johannes Schindelin thanked him, saying it helps plan the Git for Windows roadmap, and Ramsay Jones reported a successful Cygwin build of master with the experimental Rust package, with test failures identical to v2.56.0-rc2.

### git-p4 shell metacharacter injection in applyCommit
`applyCommit()` in git-p4 builds a `git diff-tree ... | git apply ...` pipeline as a shell string, so a `--commit` value like `$(some-command)` is executed by the shell. The patch replaces the pipeline with two argument-vector subprocess calls connected by a pipe. Today Anupam Mediratta posted v2, adding an `ignore_error` parameter to the new `diffTreeApply` helper so it raises `CalledProcessError` on non-zero exit by default, preserving the old behavior where a failed apply aborts the submit, while the two `tryPatch` callers pass `ignore_error=True`.

### version number after Git 2.56
The thread asks what the release after Git 2.56 should be called, with options from Git 3.0 down to 2.57. Today Harald Nordgren argued Git need not signal an upcoming 3.0 in advance and proposed changing several default configuration values before 3.0. Junio C Hamano rejected that interpretation, explaining an advance version jump warns downstream projects and distributions that something major is coming, and said Harald's proposed changes would not be part of 3.0 because no draft code or design had been presented beforehand.

## In brief
- Pushkar Singh posted [PATCH v3] fixing swapped default reflog expiry periods (reachable entries expiring after 30 instead of 90 days), splitting the regression test into four independent cases; Junio C Hamano judged the tests much better organized and answered "Probably" to fast-tracking for Git 2.56.
- Phillip Wood spotted a memory leak in [PATCH v2 4/4] "builtin/stash: merge index in-core"--the merge should be finalized before returning--and Junio C Hamano questioned whether the tree order in the `merge_incore_nonrecursive()` call is correct.
- Patrick Steinhardt posted a 7-patch series making the repository passed to `create_repository()` a pure out-parameter by calling `repo_clear()` first, eliminating the last in/out state ("core.sharedRepository").
- Patrick Steinhardt posted a 7-patch Meson improvements series speeding clean builds from ~6.8s to ~5.0s, fixing completion test failures, updating subproject wrappers, and fixing hanging GitLab MSVC jobs by unsetting the system credential.helper.
- Julia Evans posted a 7-patch series adding a `gitmergeconflicts(7)` guide; Jeff King explained a `make check-docs` failure in `seen` and suggested listing the guide in `command-list.txt`.
- Patrick Steinhardt reviewed [PATCH v2] "object-file: fix packfile flush during transaction commit" and said the version looks good to him.
