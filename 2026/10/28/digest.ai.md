# Git mailing list daily digest for 2026/10/28

## Long threads

### fetch refmap configuration in shallow repositories

*Long thread (root 2026/09/19/14-47-26): 60 replies so far from 4 participants over 40 days; 3 messages that day.*

The series "fetch: avoid fetching every branch of a new remote in a shallow repo" adds `remote.<name>.fetchRefmap` (renamed from `remote.<name>.refmap` in v12) to control where fetched refs are stored. Phillip Wood asked what should happen when `fetchRefmap` exists and a `remote.<name>.fetch` entry contains a destination; Junio C Hamano listed three alternatives--fetch destination precedence, storing in two locations, or erroring out--and noted the command-line `--refmap` behavior stores an explicitly mapped ref in both locations. Harald Nordgren said v12 appears to implement that two-location behavior and reported writing a test confirming an explicit-destination fetch entry is stored both through `fetchRefmap` and at its explicit destination, which he could include in a future v13.

### new beginner Git tutorial documentation

*Long thread (root 2026/10/06/19-37-00): 33 replies so far from 6 participants over 23 days; 3 messages that day.*

Julia Evans posted v4 of "doc: add new git tutorial", replacing the old `gittutorial` with a walkthrough from `git init` to `git push`, now more Windows-friendly: Git for Windows editor configuration during install, SSH steps marked Mac/Linux only with Git Credential Manager suggested on Windows, "folder" for "directory", "bottom left" for the vi tildes, and a link to editor configuration. Junio C Hamano flagged that "it" in "and it comes with" is ambiguous between Git Credential Manager and SSH, and on the companion "doc: remove gittutorial" patch noted the removal alone would break `make check-docs` via a dangling reference in `Documentation/git.adoc`, though he would update the topic in 'seen' as-is; he asked whether this version is in good shape for 'next'.

### repack corner cases for cruft-less MIDXs

*Long thread (root 2026/09/30/01-28-34): 29 replies so far from 5 participants over 29 days; 2 messages that day.*

Taylor Blau's series fixes cases where, with `repack.midxMustContainCruft=false`, geometric repacks and incremental MIDX writes can omit packs needed for bitmap generation even though all reachable objects remain available. Junio C Hamano questioned the topic's viability after nearly a month of mixed feedback and asked whether it had passed the point of diminishing returns; Taylor Blau stated the topic should be ready to go as-is.

### object storage extension

*Long thread (root 2026/10/19/08-55-55): 27 replies so far from 4 participants over 10 days; 2 messages that day.*

Patrick Steinhardt's series introduces the `extensions.objectStorage` extension with "<schema>[://<payload>]" syntax, supporting the "files" backend whose payload can relocate the objects directory, plus `--object-storage-format=` options for init, clone, and submodule. Karthik Nayak reviewed v4, suggested a commit-message typo fix ("s/he/the"), and said he is overall happy with the version; Patrick Steinhardt fixed the typo locally but refrained from sending a new version just for that.

### fetch advertising fetched refs when backfilling tags

*Long thread (root 2026/10/08/22-10-37): 21 replies so far from 4 participants over 21 days; 5 messages that day.*

Karthik Nayak posted v5 of "fetch: advertise fetched refs when backfilling tags", which fixes shallow fetches re-requesting objects during tag backfill by advertising already-fetched refs as 'haves' (also fixing `--atomic`); v5 adds a fix for a pre-existing remote-helper transport cleanup leak and moves the upstream-info test into its own commit. Patrick Steinhardt reviewed it with minor nits--patch 1/3 "looks obviously good", patch 3/3's test would be better ordered earlier--and Karthik Nayak said he will apply the changes locally without re-rolling unless further comments arrive.

## The day in brief

Julia Evans's v4 beginner tutorial and Karthik Nayak's v5 tag-backfill fetch fix led the day, while the object-storage extension drew a positive review and Taylor Blau declared the cruft-less MIDX repack topic ready as-is. New reports covered `check-ignore` matching trailing-slash paths against CRLF blank lines and sigchain recovery from concurrent double-pops.

## Notable threads

### sigchain recovery from concurrent double-pops
ian4hu pinged his sigchain patch, queued in 'seen' as `iy/sigchain-recover-concurrent-pops` and marked "Needs review", asking whether repair-after-the-fact is acceptable versus serializing the chain mutation. Phillip Wood diagnosed the root cause as the handler chain being unwound in two threads at once and proposed blocking signals when spawning threads plus `sigaction()`; ian4hu conceded his patch is a band-aid, and Wood called the thread-level fix probably the safest, inclined to drop the current patch if signals are handled in a single thread.

### sequencer honoring --empty when a fixup empties its target
Farid Zakaria posted v5 of "sequencer: honor --empty when a fixup!/squash! empties its target", fixing `git rebase --autosquash --empty=drop` halting when a fixup cancels out its target commit. The last fixup of a chain now checks whether HEAD was emptied even when that fixup is itself dropped as already upstream, with a new `set_fixup_target_picked()` helper recording what a "pick" left at HEAD.

### check-ignore matching trailing-slash paths against CRLF blank lines
Mike Beaton reported that with a `.gitignore` containing a single blank CRLF line, `git check-ignore -v` reports trailing-slash paths like `foo/` as ignored by an empty pattern, while LF-only blank lines and slash-less paths do not match; reproduced on Git 2.56.0 and 2.51.1, with real files inside those directories still visible to `git status` and eligible for `git add`.

### submodule foreach reverse-traversal options
William Hatfield posted v2 adding `--reverse-traversal`, `--include-superproject`, and the `--reversive` shorthand to `git submodule foreach`, visiting nested submodules before their containing repositories and the superproject last, with tests in `t/t7427-submodule-foreach-reversive.sh`.

### error reporting for repository discovery
Kaartic Sivaraam posted v3 of a five-patch series making "not a git repository" errors say how the repository was discovered: the source of an explicit git directory, the auto-discovery search range and `GIT_CEILING_DIRECTORIES`, and an unusable `GIT_OBJECT_DIRECTORY`. Junio C Hamano praised the "searched from ... up to ..." line but questioned how much the `GIT_OBJECT_DIRECTORY` message helps, saying patches 1/5-4/5 make sense while he is unsure about 5/5.

## In brief
- Patrick Steinhardt posted v3 of "odb/files: reload alternates on retry again"; Junio C Hamano noted a "Mountpoint" misspelling in the reinstated NEEDSWORK comment, asked whether the topic is ready for 'next', and offered to fix up a short list locally.
- Matt Hunter posted v3 of "init: show absolute path of (re)initialized repository", restoring the absolute path via `repo_get_git_dir()`; Patrick Steinhardt is happy with the version and Junio C Hamano marked it ready for 'next'.
- Aniket Gupta added `relative_url_gently()` so bundle-uri reports unresolvable relative URIs as errors instead of `die()`ing, updating `t/t5750-bundle-uri-parse.sh` and dropping its TODO.
- Patrick Steinhardt reviewed the loose-ref seeking fix across directory boundaries, judging it overall good but wondering whether it should be split into three patches; Junio C Hamano said they should hear from the author first.
- Patrick Steinhardt agreed to pass `OBJECT_INFO_SKIP_FETCH_OBJECT` explicitly in `is_not_in_promisor_pack_obj()` for clarity.
- D. Ben Knoble reviewed the "simple" `@{push}` triangular-workflow fix, judging the added code probably right based on the described logic.
