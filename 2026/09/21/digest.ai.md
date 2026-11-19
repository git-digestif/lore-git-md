# Git mailing list daily digest for 2026/09/21

## The day in brief

Karthik Nayak delivered a detailed review of the refs batched-deletion series, pressing on all-or-nothing semantics and backward compatibility, while Junio C Hamano weighed in on the strbuf-safe RFC with a memory-leak finding and API design questions. Jiri Kuncar posted a v2 of the `git pull` segfault fix, and Junio announced he would queue the `diff --no-index -R` fix.

## Notable threads

### refs: report old OIDs for batched deletions

This series teaches `refs_delete_refs()` to accept aligned old OIDs so the reference-transaction hook receives real values instead of zero for branch, tag, fetch, and remote deletions, restoring compare-and-delete behavior for branch and tag and preventing stale prune scans from deleting refs updated by another process. Karthik Nayak questioned patch 1/3's implicit conversion of null OIDs to NULL, suggested appending OIDs inside `collect_tags()` in patch 2/3, and warned that patch 3/3 turns pruning into an all-or-nothing failure, proposing `REF_TRANSACTION_ALLOW_FAILURE` plus a list of test and code cleanups; he also raised backward-compatibility concerns for commands like `git fetch --prune` and asked whether an LLM was used. Maciej Ciemborowicz agreed to all the suggested changes, disclosed that he used an AI coding agent while taking responsibility for the submitted version, and said he hoped to prepare v3 the next day. Junio C Hamano argued that packing all deletions into one transaction makes them all-or-nothing, contradicting the documented "attempt to keep going" semantics of `refs_delete_refs()`, and noted an unclear commit-message sentence and a misplaced BUG check in patch 1/3.

### refs: run copy and rename through transactions

This patch makes reference copy and rename operations go through the transaction API so the reference-transaction hook sees the logical updates, recording and verifying source and destination values after taking backend locks to reject concurrent changes. Junio C Hamano called the commit message "exceptionally well-written" and suggested dropping "currently" and adding "test", then questioned whether unrestricted hook-skipping via transaction flags could be avoided by rearranging the call chain, whether `refs_copy_or_rename_ref` would ever gain a third mode (suggesting `bool copy`), whether rename/copy-specific fields could stay out of the generic `ref_transaction` struct, and whether the structure implies only a single rename or copy per transaction.

### strbuf-safe: a "safe" strbuf API

This RFC series, motivated by trace2's recursive `die()` loops when strbuf allocation fails, proposes a "safe" strbuf API that returns error codes instead of dying, touching `strbuf.c`, `json-writer.c`, `wrapper.c`, and the trace2 targets. Junio C Hamano argued the longer-term goal should be making service routines like strbuf, string_list, and csum_file free of `die()`, agreed strbuf is a good first instance, and suggested renaming non-strbuf-specific functions like `strbuf_realpath()`. He also noted `safe_memory_limit_check()` reports a confusing zero limit when `git_alloc_limit` is 0, identified a memory leak in `srealloc()` where the original pointer is clobbered when `realloc()` fails, and questioned whether `enum safe_result` is an enumeration of error kinds or bitmasks, offering alternative `jw_release()` implementations for each reading plus a plain-`int` third option.

### diff --no-index: fix -R with file/directory conflicts

This patch fixes `git diff --no-index -R` misreporting paths when one side has a file where the other has a directory, by swapping the filespecs of the early queue entry when `reverse_diff` is set, with regression tests for both directions. Junio C Hamano said he will queue the fix.

### pull: avoid segfault when commit lookup fails

This v2 patch adds NULL guards in `get_can_ff()` and `already_up_to_date()` so a failed `lookup_commit_reference()` -- for example from a loose object truncated by a racing fetch or gc -- is treated as "cannot fast-forward" and "not up to date", falling through to the merge path that diagnoses the broken object instead of segfaulting. Jiri Kuncar rewrote the commit message per SubmittingPatches and changed the test to corrupt the loose object in place, since a missing object still recorded in the commit-graph is caught by fetch before pull reaches the fast-forward check.

## In brief

- Harald Nordgren's patch adds a hint when `<ref>~N` fails at a shallow boundary; Junio C Hamano questioned whether discussing `^N` helps, endorsed remote/branch naming, and asked whether the hint should also appear for `git log -20 HEAD`, which Harald noted produces no error while Junio countered that the silent stop is as puzzling as a failed `git show`.
- Antonin asked about documenting Git's governance; Junio C Hamano described the community as loosely knit self-nominated volunteers with no official reviewer list, said he would not oppose a descriptive document, but warned a prescriptive one could spark unproductive disagreement.
- Yongqiang Tian posted [PATCH v3] removing `die_lasterr()` in `compat/winansi` in favor of direct `GetLastError()` reporting at four call sites, adding Helped-by trailers for Johannes Sixt and René Scharfe and moving build validation below the separator.
- **line-range: trim trailing empty lines**: Kristofer Karlsson's patch trims trailing empty lines from `git log -L :funcname:` ranges. Junio C Hamano noted whitespace-only lines aren't treated as empty and wondered about CRLF files; Kristofer agreed "a blank line is not the same as an empty line," questioning what behavior is wanted.

- Daniel Martí's libsecret credential patch loads secrets explicitly instead of via `SECRET_SEARCH_LOAD_SECRETS`; Junio C Hamano said the ball is in the author's court to address the review questions.
- Phillip Wood reviewed the stash autostash fix series, suggesting commit-message and API changes for the in-core index merge, while Junio C Hamano called the unused-header removal a no-brainer.
- Harald Nordgren's `fetch.shallow` config narrows fetches in shallow clones to the tracked upstream; Phillip Wood questioned the scenario, and Harald confirmed it was a shallow single-branch clone that later gained a second remote.
