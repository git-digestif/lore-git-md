# Git mailing list daily digest for 2026/11/12

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 76 replies so far from 6 participants over 55 days; 4 messages that day.*

Harald Nordgren's v13 six-patch series adds `remote.<name>.fetchRefmap`, a `--limited-fetch` option for `git remote add`, defaulting to limited fetch in shallow repositories, and a sixth commit so `@{push}` resolves through `remote.<name>.fetchRefmap` for a `--limited-fetch` remote. Today Phillip Wood asked what happens when `remote.cabal.fetch` is a refspec with an explicit destination while `remote.cabal.fetchRefmap` is also set, and reiterated his objection to patch 3/6: after patch 1, `fetchRefmap` covers "where to store it" and `fetch` can cover "what to fetch", so using one variable for both is a mistake, and a limited fetch set does not need to be in the same series. He also asked which other topic the series is holding up. Matt Hunter identified the blocked topic as `ch/remote-defer-follow-remote-head-validation`, which has textual conflicts with this series, and noted that merging up to patch 3/6 could resolve it. Junio C Hamano answered the refspec question by clarifying that `fetch = master` is already a refspec, reframing the issue as what happens when a `.fetch` destination differs from what `.fetchRefmap` suggests; he noted refs are stored twice in that case even without this series, a long-standing behavior since `--refmap`, and said he is not sure it is worth tweaking, but if it were it should be a preliminary cleanup before the series.

### credential-store: add lockTimeout config option

*Long thread (root 2026/09/02/08-31-37): 41 replies so far from 5 participants over 72 days; 2 messages that day.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, retaining `lockTimeoutMS` as a deprecated alias. Today she pinged for a review of v2, which merged the zero-timeout tests into one loop, wrapped long lines in `credential-store.c`, and dropped a test that never held the lock. Junio C Hamano questioned whether the change makes sense: if all timeout variables are measured in milliseconds the suffix buys nothing and should be dropped, while if units are not uniform the config name should carry the unit, and he said the commit message must explain why the change makes sense rather than saying somebody told the author to make it.

### doc: rewrite detached HEAD state explanation

*Long thread (root 2026/10/07/14-52-01): 34 replies so far from 5 participants over 37 days; 8 messages that day.*

Julia Evans posted v6 of her rewrite of the detached HEAD explanation in `Documentation/`, a standalone guide referenced from the `advice.detachedHead` message, incorporating the agreed wording "HEAD can store either", single quotes instead of backticks in the `advice.c` message, and updated expected output in `t/t2020-checkout-detach.sh`. The discussion turned to why detached HEAD is useful: Junio C Hamano explained that in his maintainer workflow he detaches HEAD at a topic's tip to experiment and take snapshots without contaminating the branch; Julia Evans pushed back on the "developer's mindset" framing, and Junio clarified he meant the curiosity to touch and break things while sightseeing. Evans responded that the guide already states that new commits made in detached HEAD state will not be on a branch.

### root refs vs pseudorefs

*Long thread (root 2026/10/22/21-28-48): 30 replies so far from 4 participants over 22 days; 4 messages that day.*

Andy Koppe posted v4 of his five-patch documentation series distinguishing root refs from pseudorefs: the glossary stops contrasting the two, `git-ls-remote(1)` describes its default output including peeling and `--refs`, `gitdatamodel(7)` mentions the root ref naming rules and drops "so-called", and the `rev-list --merge` description and error message are reworded. He replied to earlier review by dropping the pseudoref/root-ref contrasts, correcting `--refs` to "Do not show `HEAD` or peeled tags in the output", and acknowledging that root-ref naming rules are incompletely enforced in the code. Junio C Hamano wrote that the series "looked all sensible".

### Fix segfaults on settings given without a value

*Long thread (root 2026/11/07/08-42-32): 22 replies so far from 5 participants over 6 days; 3 messages that day.*

Thomas Bachem's v3 fifteen-patch series fixes segfaults when settings such as `rebase.missingCommitsCheck`, `remote.*.tagOpt`, `color.blame.repeatedLines`, and `safe.bareRepository` are given without a value; v3 makes `tagOpt` and `repeatedLines` warn and ignore a missing value like patch 1. Today Phillip Wood reviewed the v3 range-diff and stated it is ready for `next` as far as he is concerned, Derrick Stolee found it reasonable, and Junio C Hamano marked the topic ready for `next`.

## The day in brief
Junio C Hamano marked the fifteen-patch segfault-fix series ready for `next`, and separately ejected the `--follow` speedup topic in favor of an opt-in `--follow=merges` patch that restores the pre-regression runtime on linux.git. The v13 `fetchRefmap` series drew fresh questions, with Matt Hunter naming the dependent topic it textually conflicts with.

## Notable threads

### log --follow speedup topic ejected for opt-in change
Junio C Hamano ejected the "speedup" topic and queued the opt-in patch making `--follow` following renames for non-linear history optional instead, stating that improvements to the `--follow`-merge code can be discussed separately once the opt-in change is settled.

### log: make --follow following renames for non-linear history optional
Miklos Vajna posted a patch making per-parent rename diffing opt-in: since commit 304812ed33, `git log --stat --follow` on linux.git takes 6m43s versus about 28 seconds before, and plain `--follow` now behaves as before that commit while `--follow=merges` follows the path separately in each parent of a merge, with a matching `log.follow=merges` configuration option. He implemented the optional-parameter syntax after Patrick Steinhardt leaned toward extending `--follow` rather than adding a separate flag.

### promisor-remote reply missing from fetch commands
Christian Couder posted a three-patch series fixing a protocol v2 bug where the client omits the `promisor-remote=...` reply from `fetch` commands, which works by luck over `file://` or `ssh://` but fails with stateless servers such as HTTP and JGit/Gerrit, and when fetching only object IDs. Patch 1 adds a `promisor_remote_reply` parameter to `write_command_and_capabilities()`, patch 2 makes `do_fetch_pack_v2()` send the reply, and patch 3 documents the requirement. Junio C Hamano objected to the dedicated parameter contaminating the generic interface and suggested generalizing the `server_options` list to hold keys such as `promisor-remote`.

### ref-filter: fix double free when an object is missing
Mustafa Bayar submitted a patch fixing a double free in `git for-each-ref` when a ref points to a missing object, by resetting `oi->content` at the start of `get_object()`, with a regression test in `t/t6301-for-each-ref-errors.sh`. Karthik Nayak rubber-stamped the patch, and Junio C Hamano said he will queue it and mark the topic ready for `next`.

### refs: fix reflog of a symref whose target is in the same transaction
Karthik Nayak reviewed tsukasa inoue's patch adding `ref_transaction_resolve_new_oid()` so both ref backends record the new value in a symref's reflog when the transaction also updates the target, suggesting `string_list_lookup()` instead of the linear-search helper, scheduling reflog updates at the end of the transaction, rewriting the test in a subshell, and adding `REF_TRANSACTION_ALLOW_FAILURE` tests.

## In brief
- Aniket Gupta's fsck patch reports the real object count in the "Checking connectivity" message; Laveesh Bansal noted that in a partial clone the count includes blobs never downloaded, and a title rewording was proposed.
- Adrian Ratiu posted v2 of `ls-remote --show-object-format`, printing the remote's object format; Junio C Hamano reviewed it as "Looking good" with adequate tests.
- Karthik Nayak called Patrick Steinhardt's v4 series converting `builtin/rerere` to subcommands a welcome improvement and concluded "Looks good". *Long thread (root 2026/10/27/10-46-43): 22 replies so far from 4 participants over 17 days; 1 message that day.*
- **Struct commit padding**: Kristofer Karlsson's patch moves `index` into the padding hole in `struct commit`, saving about 10% memory per commit; Karlsson proposed a compile-time assert keeping `object` at offset 0.

- Adrian Ratiu posted a three-patch `compatObjectFormat` series fixing a BUG() on matching formats, a wrong id from `rev-parse --output-object-format`, and unknown commit headers during conversion.
- Siddharth Shrimali identified the `repack --drop-filtered` abort on index-referenced blobs as a known limitation with a fix already planned: warn and drop the remaining eligible blobs.
- Karthik Nayak posted v2 of the twenty-patch series reorganizing `t9300` fast-import tests into dedicated files.
- Jakub Kulik reported that the `uname()` check in `version.c` fails on Solaris, where the syscall returns non-negative on success; Junio C Hamano confirmed the `< 0` fix is correct per POSIX and asked for a real patch.
- Srujith A Poojary posted a patch removing `USE_THE_REPOSITORY_VARIABLE` from the archival modules in favor of explicit `struct repository` pointers.
