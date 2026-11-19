# Git mailing list daily digest for 2026/10/16

## Long threads

### CI housekeeping and modernizations

*Long thread (root 2026/10/08/10-01-18): 39 replies so far from 6 participants over 9 days; 1 message that day.*

Patrick Steinhardt's eight-patch series "ci: some housekeeping and modernizations" fixes smaller CI issues and moves jobs off EOL Docker images, renaming "linux-exotic" to "linux-feature-testbed" and reworking the reftable job; earlier revisions reworked the t5004 test for SHA-256 and the series was marked for 'next'. Today Junio C Hamano argued that resurrecting the no-Rust job, dropped along with the unused "linux-clang" logic, is in the same spirit as patch 8/8 and could be added now to the linux-feature-testbed or linux32 job to ensure the system builds without Rust, though only until Rust becomes required.

### credentialStore.lockTimeout config option

*Long thread (root 2026/09/02/08-31-37): 38 replies so far from 5 participants over 45 days; 2 messages that day.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, retaining `lockTimeoutMS` as a deprecated alias, across `builtin/credential-store.c`, `Documentation/config/credential.adoc`, and `t/t0302-credential-store.sh`. Today she posted v2, which merges the two "lockTimeout=0" and "lockTimeoutMS=0" tests into a single loop over both names, wraps the long lines added in `credential-store.c`, adds a comment explaining that the "both are set" test checks for "in 0 ms" rather than "in 500 ms", and drops the "lockTimeout allows a store" test.

### Incremental connectivity check via rev-list

*Long thread (root 2026/09/14/09-47-54): 27 replies so far from 3 participants over 33 days; 1 message that day.*

Kristofer Karlsson's series adds an incremental mode for the connectivity check, gated behind `transfer.connectivityCheck=incremental`, running a verifier inside the rev-list subprocess so cost no longer grows with the number of reachable objects from the boundary. Today he benchmarked full versus incremental checks across several repositories, finding incremental up to ~2.8x faster on large and medium repos and only slightly slower on small ones, and concluded a 1000-commit cutoff is a valid strategy; he proposed three configuration modes -- "full", "incremental", and "auto" -- agreed to switch the counters to `size_t`, and agreed to a mixed approach on `die()` in library code that introduces no new calls.

### ".keep" lockfile ownership in ODB transactions

*Long thread (root 2026/10/16/02-28-58): 6 replies so far from 2 participants over 1 day; 8 messages that day.*

Justin Tobler's series fixes a bug where a `git-receive-pack` process could delete a concurrent process's ".keep" lockfile, leaving its newly written packfile vulnerable to housekeeping, by deferring lockfile creation to ODB transaction commit. Today he posted [PATCH v2 0/4], expanding the series to four patches: the hash-agnostic comment change split into its own commit, `write_pack_lock()` renamed to `write_pack_lockfile()`, improved error handling with errno reporting, and a test asserting the packfile is not migrated when a conflicting ".keep" exists.

## The day in brief
Junio C Hamano marked Patrick Steinhardt's odb/files alternates cleanups ready for 'next' and said he will queue Siddharth Shrimali's repack --dry-run fix as-is. New work included Ian Hu's sigchain fix for concurrent double-pops and Dan's bug report that token-only URL pushes fail after libcurl stopped returning credentials in `CURLINFO_EFFECTIVE_URL`. Okhuomon Ajayi posted v2 of the credentialStore.lockTimeout patch and Justin Tobler posted v2 of the ".keep" lockfile ownership fix.

## Notable threads

### sigchain recovery from concurrent double-pops
Ian Hu submitted a patch to `sigchain.c` repairing corruption from two concurrent pops of the same signal, which otherwise leaves the dispatcher stuck and the process spinning at 100% CPU until SIGKILL. The fix adds a `pushed` counter and reinstalls the pre-chain disposition `s->old[0]` when popping an already-empty chain; fault-injected A/B testing showed the unpatched code froze in 121 of 300 runs while the patched code never froze in 300 runs.

### http push fails with token-only URL after libcurl change
Dan reported that `git push` over smart HTTP fails when the remote URL carries a token-only userinfo such as `https://<token>@forge.example/...`, because libcurl 8.23.0 (CVE-2026-94266) stopped returning credentials in `CURLINFO_EFFECTIVE_URL`; fetch and clone keep working. He traced the failure to `update_url_from_redirect()` treating a credential-free effective URL as a redirect, clearing `http_auth`, and proposed comparing URLs after stripping `user[:pass]@`; the workaround is `user:token@` or a credential helper.

### repack --dry-run no longer rebuilds packs
Siddharth Shrimali posted [PATCH v3] "repack: do not rebuild packs on --dry-run", fixing `cmd_repack()` falling through into regular repack code after listing drop candidates; the patch returns early, drops two now-redundant `!dry_run` checks, and adds a test that `--dry-run` leaves the pack directory untouched with and without `-d`. Junio C Hamano reviewed it, nitpicked two commit-message sentences, and said "Will queue as-is."

### odb/files alternates cleanups
Patrick Steinhardt's six-patch series contains follow-up cleanups and improvements to the "files" object database backend now that alternates have been migrated into it, including performance fixes when reading and streaming objects with alternates and a regression test. Today Junio C Hamano marked the topic ready for 'next'.

### log --follow speedup for non-linear history

Miklos Vajna's patch speeds up `git log --follow` after a regression made a linux.git invocation take 6m43s instead of 28s, by limiting expensive rename detection to when the path is added relative to a parent and not unchanged in any parent, using `remember_follow_pathspec` when it matches a parent. Today Junio C Hamano questioned whether the `has_same` shortcut is broken in a merge where one branch updates a file in place while another renames it, concluding "That somehow sounds broken to me."

## In brief
- Julia Evans posted v4 of the detached HEAD documentation rewrite, a standalone `gitdetached-head` guide wired into `advice.c` and `command-list.txt`; Junio C Hamano reported the `advice.c` change breaks t2020 and supplied a squash patch updating the test expectations.
- Thomas Bachem posted a patch making the `sideband.allowControlCharacters` parser skip past an unrecognized keyword after warning instead of warning forever, which could hang fetch/push/clone/ls-remote on a typo; Tuomas Ahola objected to the `!!` usage, then retracted after Junio C Hamano defended it.
- Szymon Wilczek posted [PATCH v2] "cache-tree: do not die when the index is locked", reordering the commit message to lead with the symptom and changing the lock check to `fd >= 0`, so a rebase no longer dies mid-pick when a background process holds `index.lock`.
- Harald Nordgren submitted a patch making `branch.autoSetupMerge=inherit` use a remote-tracking start point directly as the upstream; Junio C Hamano corrected the documentation wording and questioned whether the same should apply to non-tracking start points.
- Patrick Steinhardt explained his design for keeping root refs out of the shared loose ref cache in the files backend, reading them into a separate cache per iteration; Junio C Hamano asked whether the topic will be ready once the open question is cleared.
- Patrick Steinhardt proposed setting `GIT_DEPTH: 0` for the GitLab CI "check-whitespace" and "check-style" jobs so they do not fail on merge requests with more than 50 commits; Junio C Hamano said he will queue the patch.
- Julia Evans posted v3 of the `man git` rewrite teaching users to navigate the docs via `git help`; Junio C Hamano queued it.
- Jeff King reported that the "refs of a shallow alternate are ignored" test fails on Windows CI with an "unable to normalize alternate object path" error and suggested using a relative path and an environment variable.
- **Stash completion feedback**: Patrick Steinhardt leaned toward dropping "stash save" completion given its 2017 deprecation, asked for clarification on NOCOMPLETE handling, and noted the "store" fix should be in the commit message.

- Rong Bao discussed the fetch `--depth=`/`--unshallow` multi-remote fix, agreeing that the outer `--depth=` is not propagated to subfetches and deciding to limit the patch scope to `--depth=` and `--shallow-since=`.
