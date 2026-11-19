# Git mailing list daily digest for 2026/09/06

## The day in brief

Elijah Newren posted v3 of a six-patch series that keeps `send-pack` from transferring the whole tree when pushing from a shallow clone, now with a tri-state `push.shallowExcludeBoundary` and new push advice. brian m. carlson weighed in on what to call the release after Git 2.56, leaning toward Git 2.97 or 2.98. Thomas Bachem refined the rerere-versus-rebase lock patch, spelling out exactly which operations should still fail after waiting.

## Notable threads

### send-pack: avoid sending the whole tree when pushing from a shallow clone

This series changes `send-pack` so that pushing from a shallow clone does not require sending the entire tree. Elijah Newren posted v3 of the six-patch series, rebased on master after `ps/odb-pluggable-pack-generation` merged, with a rewritten cover letter, fixed error messages from a suggested testcase, and new advice for retrying problematic multi-ref pushes. Patch 4/6 introduces `push.shallowExcludeBoundary` as a tri-state option (true/false/abort) that omits shallow boundary objects from generated packs, defaulting to false; patch 5/6 changes that default to true to avoid transferring and recompressing potentially gigabytes of tree data; patch 6/6 adds `advice.pushShallowBoundary`, suggesting pushing refs one at a time when a multi-ref shallow push fails after a boundary was excluded. Patches 1-3 touch `unpack-objects`, `receive-pack`, and `shallow.c` so missing or absent boundary objects produce per-ref "missing necessary objects" rejections instead of a disconnect.

### What will come after Git 2.56?

The thread asks what to name the release after Git 2.56, whose development cycle concludes around the end of this month, with options ranging from Git 3.0 to Git 2.57. brian m. carlson listed remaining considerations: forge support for SHA-256 on remaining major forges, libgit2's SHA-256 and reftable support, and a re-roll of the lowercase-only object IDs series, which he planned to send out. He argued that software not already far along on SHA-256 is likely not worth considering, noting that JGit and Gitoxide were informed at least a year ago that SHA-256 was coming in Git 3.0, and that neither should be a blocker. His gut reaction was that Git 3.0 might be the best choice, though 2.97 might be nice, or the project could be more careful with 2.95 and then skip ahead to 3.0 when ready.

### rerere: keep a background gc from killing a rebase

Thomas Bachem's patch addresses a lock race in the rerere machinery: `git rerere gc` holds `MERGE_RR.lock` while pruning `rr-cache`, and since 2.54 the auto maintenance after every commit can start one, so a rebase's next conflict can die with `LOCK_DIE_ON_ERROR` and leave later continues refusing with "you have staged changes". Today he confirmed that a failed `git rerere` at a conflict stop is recoverable by hand if the user runs `git rerere` before resolving, and corrected the assumption that an interrupted rebase is easily recoverable, noting the "you have staged changes" advice can mislead. He stated his position: keep v3's warn-and-continue behavior at a conflict, but keep failing after the wait for recordings after resolution, for explicit `git rerere`, `forget` and `clear`, and for the clears that `am` and `rebase` run; the gc itself gives up at once without waiting.

## In brief

- In the `--force-if-includes` fix for a remote-tracking ref with no reflog, Junio C Hamano questioned whether 0 is a reasonable fallback timestamp, while Aleksei Sviridkin countered that reflog entries older than 90 days do survive and that v3 argues for zero over "now" or a fixed age.
- Kristoffer Haugsbakk circulated a draft rewrite of the `--range-diff-notes` documentation for `format-patch`; Junio C Hamano proposed instead that `--no-range-diff-notes` should mean no notes are used for the range diff, mirroring `--no-notes`.
- Justin Tobler reviewed the series that writes alternates at object-database creation time, asking whether clone options like `--shared` should be restricted to the "files" backend and saying patch 7/8 looks good.
- D. Ben Knoble said he is not planning a reroll of the `dk/use-nsec-runtime` series absent strong demand or guidance on producing positive evidence of safety.
- Tyler Cipriani confirmed that HEAD's reflog is too broad for the `--force-if-includes` check in the fix for consulting the wrong ref, accepting that the option is not useful for detached HEAD.
