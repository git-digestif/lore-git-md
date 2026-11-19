# Git mailing list daily digest for 2026/09/06

## The day in brief

Elijah Newren posted v3 of a six-patch series teaching `send-pack` to avoid sending the whole tree when pushing from a shallow clone, now including a tri-state `push.shallowExcludeBoundary` option whose default changes to true. brian m. carlson laid out the remaining considerations for naming the release after Git 2.56, with his gut reaction favoring Git 2.97 or 2.98. Thomas Bachem refined his position on keeping a background `rerere gc` from killing a rebase, arguing for a warning at conflict stops but continued failure after resolution.

## Notable threads

### send-pack: avoid sending the whole tree when pushing from a shallow clone

Elijah Newren posted v3 of the six-patch series, rebased on master following the merge of `ps/odb-pluggable-pack-generation`, with a rewritten cover letter and fixed error messages. The series addresses the problem that when pushing from a shallow clone, `send-pack` may know none of the commits advertised by the receiver and ends up sending the whole tree. Patch 4/6 introduces `push.shallowExcludeBoundary` as a tri-state option (true/false/abort) to optionally omit shallow boundary objects from generated packs, and patch 5/6 changes the default to true to avoid transferring and recompressing potentially gigabytes of tree data. Patch 6/6 adds `advice.pushShallowBoundary`, suggesting pushing refs one at a time when a multi-ref shallow push fails after a boundary was excluded from the shared pack. Patches 1-3 fix error messages in `unpack-objects` and `receive-pack` and make `shallow.c` treat an absent boundary commit as the end of a traversal path rather than dying.

### What will come after Git 2.56?

brian m. carlsson listed the remaining considerations for the post-2.56 version decision: forge support for SHA-256 on the remaining major forges, updates on libgit2's SHA-256 and reftable support, and a re-roll of the lowercase-only object IDs series, which he planned to send out. He argued that software not already far along on SHA-256 is likely not worth considering, noting JGit and Gitoxide were informed at least a year ago that SHA-256 was coming in Git 3.0, and that he was not aware of serious efforts to add Rust support for new platforms. His gut reaction was that option (3), Git 2.98 or 2.97, might be the best choice, with 2.97 being nice, or alternatively being more careful with 2.95 and then skipping ahead to 3.0 when ready.

### rerere: keep a background gc from killing a rebase

Thomas Bachem refined his position on the v3 patch that makes `rerere` wait for `MERGE_RR.lock` instead of dying when a background gc holds it. He confirmed that when `git rerere` fails at a conflict stop the user can still recover everything by hand, but stressed the warning must tell the user to run `git rerere` before resolving, since resolving first removes the conflict markers and records nothing. He corrected the assumption that a rebase interrupted mid-way is easily recoverable, noting that the "you have staged changes" refusal's suggested fixes each lose something, and that this is existing 2.55 behavior. His stated position: keep v3's behavior at a conflict, but keep failing after the wait for recordings after resolution, for explicit `git rerere`, `forget` and `clear`, and for the gc, which gives up at once.

### push: fix --force-if-includes when remote-tracking ref has no reflog

The patch fixes `is_reachable_in_reflog()` reading an uninitialized timestamp when the remote-tracking ref has no reflog, which can reject a push with "remote ref updated since checkout" although nothing on the remote has changed. Junio C Hamano questioned whether 0 is a reasonable fallback for the timestamp, arguing that `gc.reflogExpire` or 90 days ago would be a more reasonable cutoff than year 1970. Aleksei Sviridkin countered that reflog entries older than 90 days do survive since expiry happens only when gc or `git reflog expire` runs, reported an A/B test in which a now-minus-90-days fallback rejects the push while zero allows the forced update, and noted that v3 argues for zero over "now" or a fixed age.

### format-patch: learn --[no-]range-diff-notes

Kristoffer Haugsbakk provided a draft rewrite of the `--range-diff-notes` documentation to clarify that the default shows the same notes in the range diff as on the patches, and clarified that "notes override behavior" means turning off all `--[no-]range-diff-notes` options, with a concrete example where patches carry four notes while the range diff carries one. D. Ben Knoble reviewed the draft, judged it a bit long but easy to follow, and admitted he had misunderstood the override example. Junio C Hamano questioned the `--no-range-diff-notes --range-diff-notes` workaround as a strange interpretation, proposing instead that `--no-range-diff-notes` should mean no notes are used for range-diff comparison, mirroring `--no-notes`. Knoble concurred that `format-patch` should use `format.notes` over `notes.displayRef` for default notes, which Haugsbakk had flagged as a suspected bug.

## In brief

- D. Ben Knoble stated he is not planning a reroll of the `dk/use-nsec-runtime` series unless there is strong demand, and that absent guidance for creating positive evidence of safety he is not sure what more to do.
- Justin Tobler reviewed the series writing alternates at object-database creation time, asking whether clone options like `--shared` should be restricted to the "files" backend, and said patch 7/8 looks good with alternates wired through `create_object_database()`.
- Nicolas Jeanmonod proposed an RFC adding a multi-valued `clean.exclude` configuration to `git clean` for persistent exclusion patterns honored even with `-x`, and asked whether it would be considered useful and consistent with `git clean`'s semantics.
- Justin Tobler confirmed that registering submodule sources was a workaround for `config_with_options()` only reading objects from `the_repository`, and approved removing the call now that objects can be read from their respective repository sources.
- Tyler Cipriani agreed to update the log message for the `--force-if-includes` patch and confirmed that HEAD's reflog is too broad for the check, with the acknowledged downside that `--force-if-includes` is not useful for detached HEAD.
- Jeff King noted the linux32 CI build uses ubuntu 20.04, out of LTS for a year, but bumping is not an option since Ubuntu and Debian dropped i386 support; he is inclined to leave it until it creates headaches.
- Kristoffer Haugsbakk noted the Linux Kernel has simplified mandatory LLM attribution to just "Assisted-by: LLM" without naming the model, since identifying specific models adds little useful information.
