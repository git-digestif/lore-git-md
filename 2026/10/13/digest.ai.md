# Git mailing list daily digest for 2026/10/13

## Long threads

### New `gitmergeconflicts(7)` manual page linked from merge-causing commands

*Long thread (root 2026/09/24/14-44-15): 61 replies so far from 6 participants over 20 days; 2 messages that day.*

Documentation work: Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page explaining merge-conflict resolution with examples, and links to it from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` docs instead of re-explaining the process in each. Today Patrick Steinhardt reviewed the page and said he is happy with the version aside from a handful of nits: inconsistent bullet punctuation, a suggestion to always verify the result of a merge, spelling out `-p` as `--patch`, and wording around zdiff3 and "ours"/"theirs". He also noted the `git-cherry-pick` patch's commit message is inaccurate because it claims to remove the merge-conflict discussion when the patch actually replaces it with a link to the new guide. Earlier, v3 of the series had been posted after Junio C Hamano's detailed reviews, which had objected to author-added `Reviewed-by` trailers and proposed rewordings that v3 incorporated.

### `uploadpack.lazyFetchTrusted` config for server-side lazy fetch control

*Long thread (root 2026/07/10/08-51-34): 37 replies so far from 3 participants over 96 days; 1 message that day.*

Server-side fetch work in `upload-pack.c`, `promisor-remote.c`, and `setup.c`: Christian Couder's series introduces the protected `uploadpack.lazyFetchTrusted` config variable so server operators can mark repositories whose promisor remotes, configuration, and hooks are trusted for lazy fetching during clone and fetch responses, addressing the failure where a server using large-object promisors cannot serve blobs in the gap between the client's filter and the promisor threshold. Today Junio C Hamano marked the topic ready for `next`. Earlier, v5 had been rebased onto Git 2.56 with documentation and test updates, and Couder had agreed with Hamano that the mostly-refactoring series was worth marking for `next` without waiting for more reviews.

### rerere waiting for `MERGE_RR.lock` outside auto maintenance

*Long thread (root 2026/09/02/08-31-37): 33 replies so far from 4 participants over 42 days; 4 messages that day.*

A fix in `rerere.c` and the maintenance path: Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) for `MERGE_RR.lock` so a rebase hitting a conflict while a background `git rerere gc` holds the lock no longer dies, and adds a hidden `--skip-locked` to `git rerere gc` that auto maintenance passes so a background gc does nothing while the lock is held. Today Bachem posted v8 with the agreed test and wording changes, Patrick Steinhardt retracted his earlier suggestion of a separate `enum rerere_gc_flags` in favor of reusing `RERERE_NOWAIT` and called v8 "in a good enough state by now", and Junio C Hamano asked whether the topic should now be marked ready for `next`. Earlier, v7 had dropped the conflict-go-on patch and adopted the lock-wait design after review.

### `git branch --delete-merged` detecting squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 30 replies so far from 5 participants over 15 days; 1 message that day.*

A change to `builtin/branch.c`: Harald Nordgren's series makes `--delete-merged` treat a branch as merged when some upstream commit since the fork point contains all of its changes, naming that commit as "landed as" in the output, and adds `--no-squashed` to skip the slower squash search. Today Junio C Hamano said that after further thought he was no longer sure of his earlier suggestions: he argued `--delete-merged` should remain a stable, strict check deleting only branches whose commits were merged and saved upstream, with a separate `--delete-irrelevant` removing branches made irrelevant by similar upstream work. Earlier, v2 had been posted with per-upstream-commit diffing shared across branches, and reviewers had debated patch-ID versus tree comparison, rename detection, and partial-clone costs.

### CI annotations naming the failing test script and line

*Long thread (root 2026/09/25/18-54-03): 25 replies so far from 4 participants over 19 days; 1 message that day.*

CI work in `t/test-lib-github-workflow-markup.sh`: Harald Nordgren's series links failure and leak annotations in GitHub Actions to the test script, showing the file and line as plain text in the annotation (direct links fail for files unchanged in the PR) and adding `--immediate` to leak-sanitizer jobs so they stop at the first failure with the sanitizer report in a log group. Today Junio C Hamano marked the topic ready for `next`. Earlier, the series had gone through revisions reverting direct file/line linking after Phillip Wood showed the links do not navigate for unchanged files, and Wood had said "Lets get this merged!".

## The day in brief
Three topics were marked ready for `next` today: the `uploadpack.lazyFetchTrusted` server config, the `history` signing options, and the CI annotation improvements. Meanwhile Junio C Hamano reopened the design of squash-merge detection in `branch --delete-merged`, and Patrick Steinhardt delivered final nits on the new merge-conflicts manual page.

## Notable threads

### Signing rewritten commits in `git history`
A change to the history-rewriting commands and replay machinery: the series adds a signing-key option to the replay API and exposes `-S`/`--gpg-sign[=<key-id>]` and `--no-gpg-sign` on `drop`, `fixup`, `reword`, `split`, and `squash`, so rewriting signed history no longer leaves the resulting commits unsigned; `commit.gpgSign` supplies the default and the last command-line option wins, matching rebase, cherry-pick, and revert. Today Junio C Hamano marked the topic ready for `next`, noting that at v3 it had already received a "looks good enough" that he had missed.

### Default `.git-blame-ignore-revs` for `git blame`

*Long thread (root 2026/09/11/23-29-44): 23 replies so far from 4 participants over 33 days; 6 messages that day.*

A change to `builtin/blame.c`: Ravi Mistry's series makes `git blame` and `git annotate` automatically use `.git-blame-ignore-revs` at the working-tree root (or `HEAD:.git-blame-ignore-revs` in bare repositories) without per-clone `blame.ignoreRevsFile` configuration. Today Mistry posted v5 reading the working-tree copy in non-bare repositories for ergonomics, then v6 after Junio C Hamano's review, passing the list string through `build_ignorelist()` into `parse_default_ignore_revs()` and setting `->util = (void *)1` with an explanatory comment; Hamano approved the v6 design as "Very cleanly done" and said he would replace the previous version unless objections arrived.

### `checkout -m` recreating original conflict labels

*Long thread (root 2026/09/30/09-48-47): 22 replies so far from 3 participants over 14 days; 3 messages that day.*

A change to the merge-ort machinery and `builtin/checkout.c`: Phillip Wood's series teaches merge-ort to write conflict labels to `.git/MERGE_LABELS` when switching to a conflicted result so `git checkout -m <path>` reuses the original labels instead of "base"/"ours"/"theirs". Today Wood posted v4, adding a patch authored by Junio C Hamano (from a lightly edited version of his earlier diff) that caches merge config settings and conflict labels once per invocation, and Hamano reviewed v4 as sensible and said he will replace the previous version.

### `read-cache` size shortcut with active conversion

*Long thread (root 2026/01/11/19-51-49): 10 replies so far from 7 participants over 276 days; 10 messages that day.*

A fix in `read-cache.c`'s `ie_modified()`: Curtis Smith's patch stops treating a working-tree size change as proof of modification when the path is subject to conversion (text/eol attributes, `core.autocrlf`, clean filters), where different bytes can convert to the same blob -- the case where `git status` reports a file modified while `git diff` and `git add` see no change. Today Smith posted v4 and v5 with the fix inlined, new `t0020-crlf.sh` tests, and benchmarks, after clarifying with Torsten Bögershausen that they are not collaborating off-list; Junio C Hamano corrected the commit message's "nothing to commit" wording and warned the new tests may hit racy-git comparison paths.

### Letting OpenSSL verify hostnames in `imap-send`
A change to `imap-send.c`: Beat Bolli's patch replaces Git's own hostname verification with OpenSSL's. Today Patrick Steinhardt reviewed it, warning that relying solely on `SSL_get_verify_result()` permits a MITM attack that strips the certificate since `X509_V_OK` can be returned without a peer certificate, so the `SSL_get_peer_certificate()` call must be retained; he also questioned the `X509_CHECK_FLAG_NO_PARTIAL_WILDCARDS` choice, asked for an introduction and a minimum-OpenSSL note, and asked whether hostname-verification tests exist.

## In brief
- SZEDER Gábor reproduced a `git log --follow` performance regression (8.8s vs 3.3s) and traced 4.45s of the runtime to `propagate_follow_pathspec_to_parent()` on second..N-th parents of merge commits.
- Rong Bao fixed `git fetch --all --depth=` and `--all --unshallow` not deepening shallow clones by passing `--depth` to the fetcher subprocess in `builtin/fetch.c`.
- Jun Wu proposed `core.indexLockTimeout` (default 1s) so commands like `add` wait for the index lock; D. Ben Knoble and Junio C Hamano questioned the cover letter and the trade-offs.
- Erik Cervin-Edin reported the Linux fsmonitor daemon deterministically missing edits in subdirectories of a renamed directory because `add_watch()` returns early on EEXIST.
- Patrick Steinhardt posted five `odb/files` alternates cleanups, including packfiles-before-loose-objects reads benchmarked 1.89x faster; Justin Tobler reviewed all five positively.
- Mark C. Chu-Carroll posted v5 of revision filtering for `git repo structure`, after Patrick Steinhardt insisted the no-args command succeed on empty repositories.
- Kristofer Karlsson discussed the incremental connectivity check, planning to experiment with removing the global repository dependency and asking about a safety-net fallback for large pushes.
