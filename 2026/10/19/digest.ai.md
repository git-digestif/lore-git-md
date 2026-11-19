# Git mailing list daily digest for 2026/10/19

## Long threads

### CI housekeeping and modernization series

*Long thread (root 2026/10/08/10-01-18): 41 replies so far from 6 participants over 12 days; 2 messages that day.*

Patrick Steinhardt's eight-patch series fixes smaller CI issues and moves jobs off EOL Docker images, renaming the `linux-exotic` job to `linux-feature-testbed` and reworking the reftable job. Earlier, v4 reworked the `t5004` test to run under both SHA-1 and SHA-256, and Junio C Hamano marked the series for `next`. Today, Steinhardt queued the resurrection of a no-Rust CI job (adding `NO_RUST` to a Makefile-based job such as `linux-feature-testbed` or `linux32`) as part of a subsequent follow-up series he is already preparing, keeping it out of the current v4.

### `--delete-merged` finding squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 33 replies so far from 6 participants over 21 days; 1 message that day.*

Harald Nordgren's two-patch series teaches `git branch --delete-merged` to treat a branch as merged when an upstream commit contains all of its changes, naming that commit as "landed as", with `--no-squashed` to skip the slower squash search. Earlier discussion covered detection tradeoffs and whether the strict meaning should stay the default. Today, Jasper Mayone reported extensive testing of v2 on generated repositories: it handled squash merges, GitHub-style squashes with new message and committer, rebase merges, renames, and reverted squashes, and kept a branch whose work only partly landed. He also found it deletes a branch that was never merged when another author made the same change upstream in an unrelated commit, and proposed enabling that looser check per repository via `--landed=squash` plus a config default. He gave performance figures (0.078s versus 0.030s for 2.56.0 on 3056 commits and 80 branches, with `--no-squashed` restoring 0.030s; in a blobless clone the first run fetched 13 blobs while `--no-squashed` fetched none), requested a "Skipping" message for branches checked out in a worktree, and added a `Tested-by`.

### Incremental connectivity check

*Long thread (root 2026/09/14/09-47-54): 28 replies so far from 3 participants over 36 days; 1 message that day.*

Kristofer Karlsson's four-patch series adds an incremental mode for the connectivity check, gated behind `transfer.connectivityCheck=incremental`, keeping a set of trusted objects shared across incoming commits so the check does not slow as reachable objects grow. Earlier, he benchmarked full versus incremental across several repositories and proposed "full", "incremental", and "auto" modes. Today, Patrick Steinhardt agreed that a 1000-commit cutoff is a good enough first iteration for the auto-mode heuristic and proposed making it configurable via `auto[=<cutoff>]`, defaulting to 1000 when omitted.

### Contributors' Summit 2026 summary and v3.0 timeline

*Long thread (root 2026/09/22/00-11-56): 20 replies so far from 9 participants over 28 days; 2 messages that day.*

This thread carries the Git Contributors' Summit 2026 summary and the Git v3.0 timeline discussion. Today, Christian Couder announced that he will attend the Gerrit User Summit on November 9-10 and an NVIDIA hackathon on November 11-13 about making partial clones a practical replacement for Git LFS across the Git/Gerrit ecosystem. Luca Milanesio said he will present his prototype of a JGit object storage backend using Apache Cassandra at the Gerrit User Summit on November 9.

## The day in brief

Patrick Steinhardt posted the seven-patch object storage extension series for the object database, while Jasper Mayone's testing of the squash-merge-aware `--delete-merged` surfaced the "irrelevant branch" case and a proposed `--landed=squash` option. Junio C Hamano also diagnosed a protocol v2 sideband violation in the shallow-fetch fix and said he would queue two small correctness patches.

## Notable threads

### Object storage extension for the ODB
Patrick Steinhardt posted a seven-patch series introducing the `extensions.objectStorage` extension, configured like the ref storage extension with `<schema>[://<payload>]` (currently only `files`, whose payload relocates `.git/objects`), plus `--object-storage-format=`, `GIT_OBJECT_STORAGE_FORMAT`, `GIT_DEFAULT_OBJECT_STORAGE_FORMAT`, `init.defaultObjectStorageFormat`, `git rev-parse --show-object-storage-format`, `git repo` `objects.storageFormat`, and propagation through `git init`, `git clone`, and `git submodule`. Karthik Nayak reviewed the early patches, asking about relative payload paths, test-matrix coverage, and the confusing similarity between `object.format` and `objects.storageFormat`.

### Faster `--follow` for non-linear history
Miklos Vajna submitted v2 of a fix for `git log --follow` slowdowns on non-linear history, where a linux.git query regressed from about 28 seconds to 6m43s after commit 304812ed33. The patch removes `propagate_follow_pathspec_to_parent()` and records the path to follow for each parent in the existing parent loop of `log_tree_diff()`, reusing the shown diff for non-merges, and adds `follow_needs_renames()` for a rename on the side a merge did not take, plus tests in `t/t4219-log-follow-merge.sh`. Vajna asked whether to split the rethink and the new tests into separate patches.

### Slow fetches from shallow repositories over v2
Harald Nordgren's patch fixes fetches from a shallow repository hanging over protocol v2, where the server walks its whole history to compute shallow info (7 minutes on a 900k-commit clone versus about a second over v0). Today, Junio C Hamano diagnosed that calling `advertise_shallow_grafts(1)` in `send_shallow_info()` breaks protocol v2 with sideband-all enabled because it uses `packet_write_fmt()` instead of the `packet_writer` API, producing a protocol violation, and added a test in `t/t5702-protocol-v2.sh`; Nordgren acknowledged the report and said he would take a look.

### Bounding reftable object record offset counts
Matthew Suozzo's patch bounds an object record's offset count by the record's remaining length in `reftable_obj_record_decode()`, where a 64-bit varint count truncates on 32-bit `size_t`. Patrick Steinhardt objected that the proposed check does not match the described overflow -- `reftable_alloc_size()` takes a `size_t` and can wrap on 32-bit platforms -- and concluded the fix should target `reftable_alloc_size()` instead; he also called the commit message "very AI-generated". Suozzo confirmed the message was AI-generated, agreed to move the Makefile change into its own patch, and said he would be happy to make the `reftable_alloc_size()` change.

### Size changes under active conversion
The v5 patch makes `git status` stop trusting a file-size change as proof of modification when conversion is active (`text`/`eol` attributes, `core.autocrlf`, a clean filter), falling through to content comparison so a CRLF rewrite no longer shows as modified. Today, Junio C Hamano agreed that placing the `would_convert_to_git()` check in `ie_match_stat()` would be a better fit, while clarifying that his objection was about avoiding unnecessary cycles rather than correctness.

## In brief
- Patrick Steinhardt said he had nothing else to add to the `repack --dry-run` fix after Junio C Hamano said he would queue the patch as-is.
- Patrick Steinhardt approved the `.keep` lockfile ownership patch's unified exit path with two minor nits; Justin Tobler queued the nits locally and may send the series up.
- Junio C Hamano called the fix for freeing uninitialized memory in `git bundle create` "obviously a sane and sensible thing to do" and said he will queue it.
- Junio C Hamano reviewed the fix reading blob config bytes as unsigned characters and said "Very nice. Will queue as-is."
- Patrick Steinhardt said he is happy with v2 of the patch making `cache-tree` not die when the index is locked.
- Andy Koppe posted v3 keeping root refs out of the loose ref cache, with a range-diff against v1 and the documentation fixes dropped to a separate patch.
- Igor Wiedler posted v3 of the connectivity-check alternates fix with test-only Windows path changes; Junio C Hamano noted the patches were not sent as replies to the v3 cover letter.
