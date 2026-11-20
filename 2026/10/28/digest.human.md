# Git mailing list daily digest for 2026/10/28

## Long threads

### fetch refmap configuration for shallow repos

*Long thread: 60 replies so far from 4 participants over 40 days; 3 messages that day.*

This series avoids fetching every branch of a new remote in a shallow repository via a `remote.<name>.fetchRefmap` setting (renamed from `refmap` in v12) that remaps where fetched refs are stored while `remote.<name>.fetch` still decides what is fetched. Today Phillip Wood asked what should happen when `fetchRefmap` coexists with a `fetch` entry carrying an explicit destination; Junio C Hamano listed three alternatives and noted the command-line `--refmap` behavior stores an explicitly mapped ref in both locations. Harald Nordgren confirmed v12 implements that two-location behavior and wrote a test showing such an entry is stored both through `fetchRefmap` and at its destination, to include in a future v13.

### new beginner git tutorial documentation

*Long thread: 33 replies so far from 6 participants over 23 days; 3 messages that day.*

Julia Evans's series replaces the old `gittutorial` with a new tutorial covering `git init` through `git push`. Today she posted v4, making the instructions more Windows-friendly: Git for Windows' editor-configuration prompt is mentioned, SSH steps are marked Mac/Linux only with Git Credential Manager suggested on Windows, and several wordings were adjusted. Junio C Hamano reviewed the patch, flagging an ambiguous "it" in "and it comes with" and asking whether the version is in good shape to merge to `next`.

### repack corner cases for cruft-less MIDXs

*Long thread: 29 replies so far from 5 participants over 29 days; 2 messages that day.*

Taylor Blau's eight-patch series fixes omissions in geometric repacks with `repack.midxMustContainCruft=false` that can make bitmap generation fail. Today Junio C Hamano questioned the topic's viability after nearly a month of mixed feedback, asking whether it was still improving or past diminishing returns; Taylor Blau stated the topic should be ready to go as-is.

### object storage extension in the object database

*Long thread: 27 replies so far from 4 participants over 10 days; 2 messages that day.*

Patrick Steinhardt's series introduces the `extensions.objectStorage` repository extension, mirroring the ref storage extension, with a `files` backend whose optional payload relocates the object directory, plus `--object-storage-format=` for init, clone, and submodule. Today Karthik Nayak reviewed v4, suggested a commit-message typo fix, and said he is overall happy with the version; Patrick Steinhardt fixed the typo locally without sending a new version.

### advertise fetched refs when backfilling tags

*Long thread: 21 replies so far from 4 participants over 21 days; 5 messages that day.*

Karthik Nayak's series fixes a shallow-clone regression where refs fetched before tag backfill were not advertised as haves, by appending fetched tips to the transport's `negotiation_include_tips`. Today he posted v5, adding a patch fixing a pre-existing leak in the remote-helper transport cleanup and moving the upstream-info test into its own commit. Patrick Steinhardt reviewed with minor nits; Karthik Nayak said he would apply them locally and not re-roll unless further comments arrive.

## The day in brief
Julia Evans posted v4 of the new beginner tutorial with Windows-friendly instructions, and Karthik Nayak posted v5 of the tag-backfill haves fix. Patrick Steinhardt's object storage extension v4 drew a positive review, while Taylor Blau declared the cruft-less MIDX repack series ready as-is.

## Notable threads

### sigchain recovery from concurrent double-pops
ian4hu's patch repairs the aftermath of two threads concurrently popping the same signal chain, which can leave a process spinning at 100% CPU. Today he pinged the thread asking whether the repair approach is acceptable or the chain mutation should be serialized; Phillip Wood diagnosed the root cause as the handler chain being unwound in two threads at once and proposed blocking signals when spawning threads plus `sigaction()`. ian4hu conceded his patch is a band-aid and offered to build the thread-level fix, which Wood called probably the safest approach while noting it would not cover threads created by external libraries.

### sequencer honoring --empty for emptied fixup targets
Farid Zakaria's patch makes `git rebase --autosquash --empty=drop` drop a commit whose `fixup!`/`squash!` empties it instead of halting. Today he posted v5, revising the series so the last fixup of a chain checks whether HEAD was emptied even when that fixup is itself dropped as already upstream, with `ALLOW_EMPTY` set unconditionally for non-last fixups or under `--empty=keep`, and with deferred cases left to a later series.

### reloading alternates on object lookup retry
Patrick Steinhardt's series restores reloading `info/alternates` when an object cannot be found, with optimizations to limit the performance cost. Today he posted v3; Christian Couder flagged a typo in patch 4/6 and said the rest looks good, and Junio C Hamano noted a "Mountpoint" misspelling in a reinstated comment and asked whether the topic is ready for `next`.

## In brief
- Kaartic Sivaraam posted v3 improving "not a git repository" errors to name the inspected directory and its source (explicit `GIT_DIR`, auto-discovery boundaries, unusable `GIT_OBJECT_DIRECTORY`); Junio C Hamano praised the search-range message and questioned the object-directory hint.
- Kristofer Karlsson proposed skipping the repository's own object directory when listing alternate refs in the receive-pack quarantine; a perf test showed push time with about 200,000 hidden refs dropping from 0.31s to 0.10s.
- Patrick Steinhardt reviewed the loose-ref seeking fix as overall good but wondered whether its three fixed bugs warrant splitting into separate patches; Junio C Hamano said they should hear from the author first.
- Aniket Gupta added `relative_url_gently()` so an unresolvable relative URI in a bundle list is reported as an error instead of dying.
- Patrick Steinhardt agreed to pass `OBJECT_INFO_SKIP_FETCH_OBJECT` explicitly in `is_not_in_promisor_pack_obj()` for clarity, per Junio C Hamano's question.
- Matt Hunter posted v3 restoring the absolute path in `git init`'s output message; Junio C Hamano marked it ready for `next`.
- William Hatfield posted v2 adding `--reverse-traversal`, `--include-superproject`, and the `--reversive` shorthand to `git submodule foreach`.
