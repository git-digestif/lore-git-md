# Git mailing list daily digest for 2026/10/19

## Long threads

### CI housekeeping and modernization series

*Long thread: 41 replies so far from 6 participants over 12 days; 2 messages that day.*

Patrick Steinhardt's eight-patch series "ci: some housekeeping and modernizations" fixes CI issues and moves jobs off EOL Docker images, including renaming "linux-exotic" to "linux-feature-testbed". Earlier, Junio C Hamano marked v4 for 'next'. Today Patrick Steinhardt queued the resurrection of a no-Rust CI job--adding NO_RUST to a Makefile-based job such as linux-feature-testbed or linux32--into a follow-up series he is preparing, not the current v4.

### branch: --delete-merged finding squash-merged branches

*Long thread: 33 replies so far from 6 participants over 21 days; 1 message that day.*

Harald Nordgren's series makes `git branch --delete-merged` treat a branch as merged when an upstream commit contains all its changes, naming it "landed as", and adds `--no-squashed` to skip the slower search. Today Jasper Mayone reported that v2 handled squash merges, GitHub-style squashes, rebase merges, renames, and reverted squashes correctly, but also deleted a branch never merged when another author made the same change upstream. He agreed the strict meaning should stay the default, proposed a per-repository `--landed=squash` option with a config default, gave performance figures, and added a Tested-by tag.

### connected: incremental connectivity check

*Long thread: 28 replies so far from 3 participants over 36 days; 1 message that day.*

Kristofer Karlsson's series adds an incremental connectivity-check mode behind `transfer.connectivityCheck=incremental`, keeping the check from slowing as reachable objects from the boundary grow. Earlier he proposed "full", "incremental", and "auto" modes with a 1000-commit cutoff. Today Patrick Steinhardt agreed 1000 commits is a good first iteration and proposed a configurable cutoff via `auto[=<cutoff>]`, defaulting to 1000.

### Contributors' Summit 2026 follow-ups and v3.0 timeline

*Long thread: 20 replies so far from 9 participants over 28 days; 2 messages that day.*

This thread follows the Git Contributors' Summit 2026 summary and Junio C Hamano's proposed v3.0 timeline. Today Christian Couder announced he will attend the Gerrit User Summit on November 9-10 and an NVIDIA hackathon on November 11-13 on partial clones replacing Git LFS. Luca Milanesio said he will present his Cassandra object-storage backend prototype for JGit at the summit.

## The day in brief
Patrick Steinhardt introduced the `extensions.objectStorage` extension for the object database, and Taylor Blau fixed the skipping negotiator to parse parents before queueing. Jasper Mayone's tests of `--delete-merged` squash detection backed the strict default while proposing a looser opt-in.

## Notable threads

### odb: introduce object storage extension
Patrick Steinhardt posted a seven-patch series adding the `extensions.objectStorage` extension, configured like the ref storage extension as "<schema>[://<payload>]"; only "files" is supported, with a payload that can relocate ".git/objects". It adds `--object-storage-format=` to git-init and git-clone, GIT_OBJECT_STORAGE_FORMAT, `init.defaultObjectStorageFormat`, `git rev-parse --show-object-storage-format`, git-repo's "objects.storageFormat", and submodule propagation. Karthik Nayak reviewed the first four patches with questions on payload paths and test coverage.

### log: speed up --follow for non-linear history
Miklos Vajna submitted v2 fixing `git log --follow` slowdowns on linux.git after commit 304812ed33 by removing `propagate_follow_pathspec_to_parent()` and recording the path to follow per parent in the existing `log_tree_diff()` loop, reusing the shown diff for non-merges. It adds `follow_needs_renames()` for a rename on the side a merge did not take, plus tests in t/t4219-log-follow-merge.sh.

### negotiator/skipping.c: parse parents before queueing
Taylor Blau's patch parses each unseen parent before queueing it in `negotiator/skipping.c`, so the date-sorted queue uses the real commit date instead of zero and old ref tips no longer precede a recent common ancestor. A new test in t/t5552-skipping-fetch-negotiator.sh verifies the ordering.

## In brief
- Junio C Hamano found Harald Nordgren's shallow-fetch fix breaks protocol v2 with sideband-all because `advertise_shallow_grafts(1)` uses the v0 packet API; Nordgren acknowledged it.
- Junio C Hamano said the `would_convert_to_git()` check for the CRLF status bug fits better in `ie_match_stat()`, and clarified his objection was about wasted cycles, not correctness.
- Andy Koppe posted v3 keeping root refs out of the loose ref cache via a temporary cache, with a range-diff against v1 and doc fixes split out.
- Patrick Steinhardt said Matthew Suozzo's reftable fix should target `reftable_alloc_size()` instead, and called the commit message overly AI-generated; Suozzo agreed to rework.
- Junio C Hamano proposed honoring config file permission bits via access(2), refusing writes to read-only files, with a POSIXPERM test.
- **status.compareBranches default**: Harald Nordgren proposed making `@{upstream} @{push}` the default for `status.compareBranches`; Junio C Hamano requested commit-message and test improvements.

- Patrick Steinhardt asked for a cover letter on Beat Bolli's v3 imap-send series and a split of the OpenSSL hostname-verification patch; Xiangfei Ding added a Reviewed-by.
