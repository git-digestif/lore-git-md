# Git mailing list daily digest for 2026/09/16

## Long threads

### http: add http.sslVerifyStatus to check stapled OCSP responses

*Long thread: 22 replies so far from 4 participants over 37 days; 1 message that day.*

This thread concerns `http.c` and the libcurl TLS options Git sets: Git never sets `CURLOPT_SSL_VERIFYSTATUS`, so libcurl never requests the OCSP "Certificate Status Request" extension and any stapled response a server sends is ignored, including one stating the certificate has been revoked. The series adds an `http.sslVerifyStatus` boolean that maps to that option, defaulting to false because enabling it would block connections to legitimate servers that do not staple OCSP responses. Earlier iterations reworked the commit message and documentation and combined the two test approaches: cases needing no OCSP setup stay in `t5551`, while everything requiring a responder lives in the new `t5585-http-ssl-ocsp.sh` with `LIB_HTTPD_OCSP` support in lib-httpd. Today Junio C Hamano said the updated tests look good and that he will replace the existing tests with them.

## The day in brief

Junio C Hamano said he will queue Orestis Floros's fix for parsing commits in the caller's repository during superproject/submodule merges, marked the pathspec-prefix v4 series ready for `next`, and approved the OCSP test updates; Git v2.56.0-rc1 was also announced for testing.

## Notable threads

### commit-reach: parse commits in the given repository

`can_all_from_reach()` and `can_all_from_reach_with_flag()` in `commit-reach.c` parsed the commits they walk in `the_repository` even when the caller worked in a different repository, which made merging a superproject whose submodule pointer advanced on both sides die with "fatal: invalid commit position" when commit-graphs existed in both repositories. Orestis Floros posted a patch passing a `struct repository *` into both functions so commits are parsed in the caller's repository, with a regression test in `t/t6437-submodule-merge.sh`. Kristofer Karlsson confirmed the new tests reproduce the bug with and without the code change and noted it reduces `the_repository` references in `commit-reach.c` from 17 to 11. Junio C Hamano called the change reasonable and a welcome step for libification, and said he will queue it.

### dir: fix pathspec prefixes with exclusions

This v4 series fixes the pathspec prefix optimization in `dir.c`: the prefix is derived from non-exclude items, so applying it while matching an exclude item can compare the wrong portions of paths, and an exclude item first in the pathspec prevents finding a common prefix among the rest. The first patch matches exclude items against the full pathname; the second starts the common-prefix search at the first non-exclude item. Today Junio C Hamano said this round looks ready for `next`.

### upload-pack: swap wanted-ref/shallow-info responses

When a server enables `uploadpack.allowRefInWant`, `upload_pack_v2()` in `upload-pack.c` sent wanted-ref info before shallow-info, but the fetch-pack client expects shallow-info first, so protocol v2 shallow fetches against such servers exited with "fatal: expected 'packfile', received 'shallow-info'". Royce Remer posted v2 swapping the send order so shallow-info goes first, a server-side-only change compatible with existing clients, adding shallow clone and incremental shallow fetch tests to `t/t5703-upload-pack-ref-in-want.sh`. Junio C Hamano agreed the explanation of the packet sequence and client breakage was good commit-log material.

## In brief

- Qin ShiCheng clarified that the repack ".keep" production failure involved pushes seconds apart rather than concurrent, said the patch only removes a ".keep" the process created itself, and announced he will drop 1/6 from v2 so the series covers only the repack side.
- Junio C Hamano announced Git v2.56.0-rc1, comprising 721 non-merge commits since v2.55.0 from 90 people, 37 of them new.
- D. Ben Knoble and Tyler Cipriani reviewed the v5 `--force-if-includes` fast-forward patch; Cipriani is open to renaming a variable and wants more feedback before rerolling.
- Phillip Wood proposed two fix directions for the `stash.index=true` autostash bug that leaves a redundant stash entry after a fast-forward merge; Ben Knoble said he would try patches.
- Yongqiang Tian posted a fix for `die_lasterr()` in `compat/winansi.c`, which passed its `va_list` to `die_errno()` as an ordinary argument; Johannes Sixt and René Scharfe debated simpler designs.
- Brigham Campbell posted v2 of `git-contacts` reading patch contents from stdin when argv is empty and stdin is not a terminal; Junio C Hamano floated a "-" filename alternative.
- Junio C Hamano proposed a `setup.c` patch exporting `GIT_WORK_TREE` for hooks in worktrees where `GIT_DIR` is also set, flagging a NEEDSWORK comment in the hunk.
- Christian Couder endorsed proposing the "Implement promisor remote fetch ordering" project for Outreachy's December 2026 cohort.
