# Git mailing list daily digest for 2026/11/12

## Long threads

### fetch: remote.<name>.fetchRefmap and limited fetches in shallow repos

*Long thread: 76 replies so far from 6 participants over 55 days; 4 messages that day.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository, adding the `remote.<name>.fetchRefmap` config and `--limited-fetch` for `git remote add`. Today Phillip Wood reiterated his objection to patch 3/6 -- after patch 1 the refmap covers "where to store it", so `fetch` can cover "what to fetch" -- and asked which topic is blocked. Matt Hunter identified `ch/remote-defer-follow-remote-head-validation` as the blocked topic, whose textual conflicts merging up to patch 3/6 could resolve. Junio C Hamano answered Wood's refspec question: a `.fetch` entry whose destination differs from the refmap's stores the ref twice, a long-standing behavior since `--refmap` was introduced.

### credential-store: lockTimeout config option

*Long thread: 41 replies so far from 5 participants over 72 days; 2 messages that day.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred name for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias in `builtin/credential-store.c`. Today she pinged for a review of v2, which merged the zero-timeout tests into one loop and dropped a test that never held the lock. Junio C Hamano questioned the rename: if all timeouts are in milliseconds the suffix buys nothing, and if units are not uniform the name should carry the unit; the commit message must explain why the change makes sense.

### doc: rewrite of the detached HEAD explanation

*Long thread: 34 replies so far from 5 participants over 37 days; 8 messages that day.*

Julia Evans is rewriting the detached HEAD explanation into a standalone `git help detached-head` guide. Today she posted v6 with the agreed wording ("HEAD can store either"), single quotes in the `advice.c` message, and updated `t/t2020-checkout-detach.sh` expectations. Junio C Hamano described why detached HEAD helps his maintainer workflow -- experimenting at a branch tip without contaminating the topic branch -- while Julia Evans pushed back on the "developer's mindset" framing, noting the guide already says detached commits will not be on a branch.

### root refs vs pseudorefs documentation

*Long thread: 30 replies so far from 4 participants over 22 days; 4 messages that day.*

Andy Koppe's five-patch series stops calling root refs pseudorefs across the glossary, `git-ls-remote(1)`, `gitdatamodel(7)`, `revision.c` and tests. Today he posted v4, dropping the pseudoref/root-ref contrasts, stating the per-worktree ref prefix list will not be extended, and rewording the `rev-list --merge` error message. Junio C Hamano wrote that the series "looked all sensible".

### Fix segfaults on settings given without a value

*Long thread: 22 replies so far from 5 participants over 6 days; 3 messages that day.*

Thomas Bachem's 15-patch series fixes segfaults when a config setting is given without a value, covering `rebase.missingCommitsCheck`, `remote.*.tagOpt`, `color.blame.*`, bundle lists and more, each with a regression test. Today Phillip Wood called the v3 range-diff ready for `next`, Derrick Stolee found it reasonable, and Junio C Hamano marked the topic ready for `next`.

## The day in brief
Junio C Hamano marked the 15-patch NULL-config segfault fix series ready for `next` and queued the opt-in `--follow=merges` patch in place of the ejected `--follow` speedup topic. Christian Couder posted a fix for the missing `promisor-remote` reply in `fetch` protocol v2, which breaks stateless servers such as Gerrit.

## Notable threads

### log: --follow rename following for non-linear history
Junio C Hamano ejected the "speedup" topic and queued the opt-in patch making `--follow` rename following for non-linear history optional instead, saying `--follow`-merge improvements can be discussed separately once the opt-in change is settled.

### promisor-remote reply missing from fetch
Christian Couder posted a three-patch series fixing a "promisor-remote" protocol v2 bug where the client omits the `promisor-remote=...` reply from `fetch`, failing with stateless servers (HTTP, JGit/Gerrit) and when fetching only object IDs. Patch 2/3 makes `fetch` send the reply; patch 3/3 documents it. Junio C Hamano objected to patch 1/3's dedicated parameter in `write_command_and_capabilities()`, suggesting the existing `server_options` list be generalized to hold keys like "promisor-remote".

### ref-filter: double free when an object is missing
Mustafa Bayar fixed a double free in `git for-each-ref` when a ref points to a missing object by resetting `oi->content` in `get_object()`, with a regression test in `t/t6301-for-each-ref-errors.sh`. Karthik Nayak rubber-stamped it, and Junio C Hamano said he will queue it and mark the topic ready for `next`.

## In brief
- Miklos Vajna posted a patch making per-parent rename diffing for `--follow` opt-in: plain `--follow` behaves as before 304812ed33, while `--follow=merges` follows each parent of a merge, with a matching `log.follow=merges` config.
- Kristofer Karlsson proposed moving `index` into the padding hole in `struct commit` (`commit.h`), shrinking it from 80 to 72 bytes on 64-bit platforms; he also proposed a compile-time assertion that `object` stays at offset 0.
- Adrian Ratiu posted v2 of `ls-remote --show-object-format`, printing the remote's object format as `object-format: <format>`; Junio C Hamano reviewed it and said "Looking good".
- Aniket Gupta's fsck patch reports the real object count in the "Checking connectivity" message; Laveesh Bansal noted partial clones count blobs never downloaded, and Gupta proposed retitling the patch accordingly.
- Jakub Kulik reported that Solaris `uname()` returns non-negative on success, breaking the `if (uname(...))` check in `version.c`; he proposed `< 0`, which Junio C Hamano confirmed matches POSIX.
