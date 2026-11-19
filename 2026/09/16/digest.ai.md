# Git mailing list daily digest for 2026/09/16

## Long threads

### http: add http.sslVerifyStatus to check stapled OCSP responses

*Long thread (root 2026/08/11/17-02-00): 22 replies so far from 4 participants over 37 days; 1 message that day.*

This thread concerns TLS certificate revocation checking in Git's HTTP transport. Git never sets `CURLOPT_SSL_VERIFYSTATUS`, so libcurl never requests the OCSP "Certificate Status Request" extension and any stapled response a server sends is ignored, including responses stating the certificate has been revoked. The series adds an `http.sslVerifyStatus` boolean that maps to that libcurl option, defaulting to false because enabling it would block connections to legitimate servers that do not staple OCSP responses. Earlier revisions reworked the commit message, added `curl_easy_strerror()` output to the die() message, and split the tests: cases needing no OCSP setup stay in t5551 while everything requiring a responder moved to a new t5585 with test CA and OCSP responder infrastructure, and v7 was posted as the combined version discussed with Junio C Hamano and Patrick Steinhardt. Today Junio C Hamano stated that the updated tests in the patch look good and that he will replace the existing tests with them.

## The day in brief

Git v2.56.0-rc1 was tagged and announced, comprising 721 non-merge commits since v2.55.0 from 90 contributors. A fix for the fatal "invalid commit position" error when merging superprojects with submodule commit-graphs was verified to reproduce the bug and will be queued, and the pathspec-prefix exclusion fix in dir.c was declared ready for `next`.

## Notable threads

### Git v2.56.0-rc1 release candidate

Junio C Hamano announced that Git v2.56.0-rc1 is now available for testing at the usual places, comprising 721 non-merge commits since v2.55.0 contributed by 90 people, 37 of whom are new. The tarballs are at kernel.org and the `v2.56.0-rc1` tag is available in the listed public repositories.

### commit-reach: parse commits in the given repository

Orestis Floros posted a patch passing a `struct repository *` into `can_all_from_reach()` and `can_all_from_reach_with_flag()` so commits are parsed in the caller's repository rather than always in `the_repository`. This fixes a fatal "invalid commit position" error (or silent wrong commit data) when merging a superproject whose submodule pointer advanced on both sides with commit-graphs present in both repositories, and adds a regression test in t/t6437-submodule-merge.sh. Kristofer Karlsson confirmed he ran the new tests with and without the code change and verified they reproduce the bug, and noted the change reduces `the_repository` references in commit-reach.c from 17 to 11. Junio C Hamano called the change reasonable and a welcome step for recent libification efforts, and said he will queue it.

### dir: fix pathspec prefixes with exclusions

The v4 series fixes pathspec prefix optimization in dir.c: exclude items are now matched against the full pathname, and the common prefix is computed starting from the first non-exclude item so an exclude item at the start no longer prevents recovering the shared directory. Today Junio C Hamano stated that this round of the series looks ready for `next`.

### upload-pack: swap wanted-ref/shallow-info responses

Royce Remer posted v2 of a server-side fix in upload-pack.c: `upload_pack_v2()` now sends shallow-info before wanted-ref info, so protocol v2 shallow fetches against servers with `uploadpack.allowRefInWant` no longer exit with "fatal: expected 'packfile', received 'shallow-info'". The patch adds shallow clone and incremental shallow fetch test cases to t/t5703-upload-pack-ref-in-want.sh. Junio C Hamano agreed that the explanation of how the packet sequence is produced and causes reliable client breakage was good material for the commit log message.

### repack: don't lose objects to a ".keep" that appears mid-run

This series addresses a production data-loss bug where `git repack -d` can delete a pack whose objects were never copied anywhere, because a concurrent push links a ".keep" file onto an existing pack between repack's scan and pack-objects' scan. Today Qin ShiCheng clarified that the production failure involved pushes seconds apart rather than truly concurrent, explained that the patch's aim is narrower than collision handling--only removing a ".keep" the process created itself--endorsed creating the ".keep" at ODB commit time as the better shape, and announced he will drop 1/6 from v2 so the series covers only the repack side.

## In brief

- D. Ben Knoble reviewed the v5 `--force-if-includes` fast-forward patch, flagging "push force" wording and commit-message style; Tyler Cipriani acknowledged the critiques and wants to gather more feedback before rerolling.
- Phillip Wood proposed two fix directions for the `stash.index=true` bug that leaves a redundant stash entry after an autostash fast-forward merge; Ben Knoble said he would try to make and test patches.
- A post-checkout hook in a worktree sees `GIT_DIR` but not `GIT_WORK_TREE`, so `git rev-parse --show-toplevel` returns the CWD; Junio C Hamano proposed a setup.c patch to export `GIT_WORK_TREE` in that case, flagging a NEEDSWORK comment in the hunk.
- Yongqiang Tian fixed `die_lasterr()` in compat/winansi.c, which had passed its va_list to `die_errno()` as an ordinary argument; Johannes Sixt and René Scharfe debated simpler single-string and macro designs.
- Brigham Campbell posted v2 of git-contacts reading patch contents from stdin when argv is empty and stdin is not a terminal; Junio C Hamano offered a "-" filename design as food for thought.
- Christian Couder endorsed proposing the "Implement promisor remote fetch ordering" project for Outreachy's December 2026 cohort.
- Junio C Hamano's "What's cooking" notes that Git 2.56-rc1 has been tagged, with about 87 non-merge commits on `next` and 166 on `seen`.
