# Git mailing list daily digest for 2026/09/21

## The day in brief

Reviewers pressed a refs batch-deletion series over its all-or-nothing pruning semantics, and the author said he hoped to prepare a v3 the next day. Junio C Hamano dissected an RFC "safe" strbuf API aimed at trace2 die() recursion and found a memory leak in its allocator. Jiri Kuncar posted a v2 of a git pull segfault fix for unparseable merge heads.

## Notable threads

### refs: supply old OIDs for batched deletions

The series teaches `refs_delete_refs()` to accept aligned old OIDs so the reference-transaction hook receives real values instead of zero for branch, tag, fetch and remote deletions, restoring compare-and-delete behavior for branch and tag and preventing stale prune scans from deleting refs updated by another process. Karthik Nayak reviewed all three patches: he questioned the implicit null-OID conversion in patch 1/3, suggested appending OIDs inside `collect_tags()` in patch 2/3, and warned that patch 3/3 turns pruning into an all-or-nothing failure, proposing `REF_TRANSACTION_ALLOW_FAILURE` plus test and formatting cleanups, and asked whether an LLM generated the patches. Maciej Ciemborowicz agreed to those changes, disclosed that he used an AI coding agent and takes responsibility for the submitted version, and said he hoped to prepare v3 the next day. Junio C Hamano added that packing all deletions into one transaction contradicts the documented "attempt to keep going" semantics of `refs_delete_refs()`, so per-ref transactions or a best-effort mode would be needed, and noted the refname/old-OID count BUG check should come after the empty-refnames early return.

### "safe" strbuf API for trace2

The RFC proposes a `strbuf-safe` API because trace2 uses json-writer, which uses strbuf, and an allocation failure there can trigger recursive `die()` loops. Junio C Hamano argued the longer-term goal should be making service routines like strbuf, string_list and csum_file free of `die()`, agreed strbuf is a good first instance, and suggested renaming functions that merely use a strbuf but are not strbuf-specific. He also noted `safe_memory_limit_check()` reports a confusing limit when `git_alloc_limit` is 0, identified a memory leak in `srealloc()` where the original pointer is clobbered before the failure check, and questioned whether `enum safe_result` means distinct error kinds or bitmasks, offering alternative `jw_release()` implementations for each reading plus a plain-`int` third option.

### pull: avoid segfault when commit lookup fails

`get_can_ff()` and `already_up_to_date()` pass the result of `lookup_commit_reference()` unchecked, so a truncated loose object behind HEAD or a merge head makes `git pull` segfault. Jiri Kuncar posted v2 adding NULL guards that treat a failed lookup as "cannot fast-forward" and "not up to date", falling through to the normal merge path which diagnoses the broken object and fails cleanly. The commit message was rewritten per SubmittingPatches, and the test now corrupts the loose object in place with `rm -f` and `echo garbage >` rather than truncating it, because a missing object still in the commit-graph is caught by fetch before pull reaches the fast-forward check; the redundant `git fetch` was dropped since `git clone` already populates the remote refs.

## In brief

- Junio C Hamano said he will queue Haokai Ding's fix for `diff --no-index -R` misreporting file/directory conflicts.
- On the refs copy/rename transaction patch, Junio C Hamano called the commit message exceptionally well-written and asked whether hook-skipping flags, a `bool copy` parameter, and rename-specific fields in the generic `ref_transaction` struct could be avoided, and whether one transaction can hold multiple renames.
- Junio C Hamano reviewed Harald Nordgren's shallow-clone advice for `<ref>~N`, endorsing the remote/branch naming and asking whether hints should also cover `git log -20 HEAD`; Harald Nordgren replied that command produces no error, and Junio countered the hint would go to the terminal where silently stopping early is puzzling.
- Kristofer Karlsson's patch trims trailing empty lines from `git log -L :funcname:` ranges; Junio C Hamano noted it treats only newline-only lines as empty, and Kristofer agreed blank and empty lines differ and the docs specify neither.
- Yongqiang Tian posted v3 of the `compat/winansi` `die_lasterr()` fix, removing the helper and reporting `GetLastError()` directly at four call sites, with Helped-by trailers for Johannes Sixt and René Scharfe.
- Asked about documenting Git governance, Junio C Hamano described the community as loosely knit self-nominated volunteers with no official reviewer list, said he would not oppose a descriptive document but warned a prescriptive one could spark unproductive debate.
- On the credential/libsecret explicit secret-loading patch, Junio C Hamano said the ball is in the author's court to address the review questions.
