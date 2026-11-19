# Git mailing list daily digest for 2026/10/16

## Long threads

### CI housekeeping and modernization series

*Long thread: 39 replies so far from 6 participants over 9 days; 1 message that day.*

Patrick Steinhardt's eight-patch series cleans up Git's CI configuration, fixing smaller issues and moving jobs off EOL Docker images, including renaming the "linux-exotic" job to "linux-feature-testbed" and reworking the reftable job. Earlier revisions addressed Jeff King's review points on the t5004 SHA-256 test, locale handling on the 32-bit job, and the reftable coverage question, and Junio C Hamano marked v4 for `next`. Today Junio C Hamano argued that resurrecting the no-Rust build job is in the same spirit as patch 8/8 and could be added now to the `linux-feature-testbed` or `linux32` job to ensure the system builds without Rust, since it is not too big an effort, though only until Rust becomes required.

### credentialStore.lockTimeout config option

*Long thread: 38 replies so far from 5 participants over 45 days; 2 messages that day.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries in `builtin/credential-store.c`, retaining `lockTimeoutMS` as a deprecated alias, with tests in `t/t0302-credential-store.sh`. Earlier, Junio C Hamano had suggested looping the two timeout tests over both variable names and questioned the "allows a store" test. Today Okhuomon Ajayi posted v2, which turns the two "lockTimeout=0" and "lockTimeoutMS=0" tests into a single loop over both names, wraps the long lines in `credential-store.c`, adds a comment explaining that the "both are set" test checks for "in 0 ms" rather than "in 500 ms", and drops the "lockTimeout allows a store" test because nothing holds the lock there.

### Incremental connectivity check

*Long thread: 27 replies so far from 3 participants over 33 days; 1 message that day.*

Kristofer Karlsson's four-patch series adds an incremental mode for the connectivity check in `connected.c`, gated behind `transfer.connectivityCheck=incremental`, to keep the check from slowing down as the number of reachable objects from the boundary grows; the verifier runs inside the existing `rev-list` subprocess via a new `--verify-trees-incremental` flag. Earlier, Patrick Steinhardt reviewed the documentation, perf tests, and implementation, raising points about `die()` in library code, counter types, and a possible fallback beyond N commits. Today Kristofer Karlsson benchmarked full vs. incremental checks across several repositories, finding incremental up to ~2.8x faster on large and medium repos and only slightly slower on small ones, concluding a 1000-commit cutoff is valid; he proposed three config modes ("full", "incremental", "auto"), agreed to switch counters to `size_t`, and agreed to a mixed approach on `die()` that introduces no new calls while accepting existing ones.

### ".keep" lockfile ownership in ODB transactions

*Long thread: 6 replies so far from 2 participants over 1 day; 8 messages that day.*

Justin Tobler's series fixes a bug in `object-file.c` where a `git-receive-pack` process could delete another concurrent process's ".keep" lockfile for an identical packfile, leaving its packfile vulnerable to housekeeping; the fix defers lockfile creation to ODB transaction commit. Earlier today Patrick Steinhardt reviewed all three patches, suggesting a `write_pack_lockfile()` rename, errno-based error messages, and a test assertion that the ".pack" file is not migrated. Later today Justin Tobler posted v2 as a four-patch series, splitting the hash-agnostic comment change into its own commit, applying the rename and error-handling improvements, and adding the test assertion.

## The day in brief
Justin Tobler posted v2 of the fix for ".keep" lockfile ownership in ODB transactions, and Kristofer Karlsson backed a three-mode `transfer.connectivityCheck` configuration with fresh benchmarks. Okhuomon Ajayi posted v2 of the `credentialStore.lockTimeout` patch, and Ian Hu submitted a fix for a sigchain race that could spin a process at 100% CPU.

## Notable threads

### sigchain concurrent double-pop recovery
Ian Hu submitted a patch to `sigchain.c` fixing a race where two concurrent handlers popping the same signal both read the same chain depth, leaving the dispatcher stuck on a popped handler and spinning at 100% CPU until SIGKILL. The patch adds a `pushed` counter and reinstalls the pre-chain disposition when popping an already-empty chain; fault-injected A/B testing showed the unpatched code froze in 121 of 300 runs while the patched code never froze in 300 runs.

### HTTP push failure with token-only URL userinfo
Dan reported that `git push` over smart HTTP fails when the remote URL carries a token-only userinfo such as `https://<token>@forge.example/...`, after libcurl 8.23.0 stopped returning credentials in `CURLINFO_EFFECTIVE_URL`; fetch and clone keep working. He traced the failure to `update_url_from_redirect()` treating a credential-free effective URL as a redirect, clearing `http_auth`, and proposed comparing URLs after stripping `user[:pass]@`; a workaround is `user:token@` or a credential helper.

### repack --dry-run rebuilds packs
Siddharth Shrimali posted v3 of a fix to `builtin/repack.c` for `git repack --drop-filtered --dry-run`, which was documented to list droppable objects without rebuilding packs but instead wrote a new promisor pack and deleted old ones. The patch returns right after listing candidates, drops two now-redundant `!dry_run` checks, and adds a test that the pack directory is untouched with and without `-d`; Junio C Hamano said he will queue it as-is.

## In brief
- Junio C Hamano marked Patrick Steinhardt's "odb/files: cleanups and improvements for alternates" topic ready for `next`.
- Junio C Hamano questioned whether Miklos Vajna's `has_same` shortcut in the `log --follow` speed-up is broken in a merge scenario where one branch updates a file in place while another renames it.
- Julia Evans posted v4 of the detached HEAD documentation rewrite, and Junio C Hamano reported the `advice.c` change breaks `t2020` without a test update, supplying a squash patch.
- Jeff King reported the "refs of a shallow alternate are ignored" test failing on Windows CI with an alternate-path normalization error and suggested a relative path fix.
- Patrick Steinhardt proposed setting `GIT_DEPTH: 0` for the GitLab CI "check-whitespace" and "check-style" jobs so they work on merge requests over 50 commits; Junio C Hamano said he will queue the patch.
