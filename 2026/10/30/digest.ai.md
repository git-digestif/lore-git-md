# Git mailing list daily digest for 2026/10/30

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 66 replies so far from 5 participants over 42 days; 2 messages that day.*

Harald Nordgren posted v13 of the series that keeps a shallow repository from fetching every branch of a newly added remote, built around the `remote.<name>.fetchRefmap` configuration. The new sixth commit, "remote: resolve @{push} through remote.<name>.fetchRefmap", gives `tracking_for_push_dest()` the same fallback to `remote.<name>.fetchRefmap` that `get_ref_map()` uses on the fetch side, so `@{push}` resolves for a `--limited-fetch` remote; the first commit adds a regression test confirming that an explicit-destination `remote.<name>.fetch` entry is stored both at that destination and through the refmap. Earlier, Phillip Wood and Junio C Hamano had pressed on the interaction between `remote.<name>.fetch` and the refmap, and Harald had confirmed the two-location behavior with a test.

### branch: --delete-integrated for squash- and rebase-merged branches

*Long thread (root 2026/09/29/07-30-30): 51 replies so far from 6 participants over 32 days; 3 messages that day.*

Harald Nordgren posted v5 of the series adding `git branch --delete-integrated <pattern>`, which deletes branches whose changes landed upstream via squash or rebase merges, including after upstream changed the same files, and names the landing commit in output like "landed as 9f8e7d6". The v5 changes are that `git branch -h` usage lines list `--delete-integrated` and that combining it with `--delete-merged` dies with a short "cannot be used together" message. Junio C Hamano reviewed patches 3/4 and 4/4, suggesting a `diffopt` rename nit and questioning whether the index has been read before `ll_merge()` is called in `merge_keeps_upstream()`.

### doc: replace gittutorial with a new beginner tutorial

*Long thread (root 2026/10/06/19-37-00): 38 replies so far from 6 participants over 25 days; 6 messages that day.*

Julia Evans posted v5 of the two-patch series that removes the old `gittutorial` and adds a new beginner tutorial; v5 replaces "it" with "GCM" in the commit message and removes trailing whitespace. Junio C Hamano squashed fixups into the series commenting out `gittutorial` in `Documentation/Makefile` and `command-list.txt` so each patch builds, then supplied a fuller fixup removing remaining references from `git.adoc`, `gitcore-tutorial.adoc`, `gitcvs-migration.adoc`, `gitglossary.adoc`, `gittutorial-2.adoc`, `gitworkflows.adoc`, and the build files, suggesting the two patches be squashed into one once contents settle. Julia Evans also reported testing the Windows Git Credential Manager login flow and explained she left old `gittutorial` references in place to avoid merge conflicts.

### Documentation: community voting process

*Long thread (root 2026/10/06/18-05-36): 29 replies so far from 13 participants over 25 days; 8 messages that day.*

Discussion of Emily Shaffer's proposed `CommunityVoting.adoc` continued. Toon Claes argued eligible voters should be able to abstain and that voting should be reserved for spread opinions; Emily Shaffer summarized reroll changes (subject "Specify...", an explicit secret-vote requirement, and votes referencing the proposal's Message-ID after Junio C Hamano asked for Message-ID rather than commit object name) and said she cannot respond until around November 12th-13th. Jeff King clarified his concern was scaling voter-roll management, proposing a human-input exception where Junio or the PLC could designate additional voters; Jonathan Corbet reported using CIVS for TAB elections.

### loose: fix migrating "loose-object-idx" with batch fsync

*Long thread (root 2026/10/22/23-40-07): 22 replies so far from 4 participants over 9 days; 9 messages that day.*

Justin Tobler's v3 series fixes ODB transactions failing on commit under a compatibility hash by merging the temporary-directory "loose-object-idx" mapping into the main one and writing it atomically. Patrick Steinhardt reviewed v3 and said the changes look exactly as expected and he is happy with the version. Karthik Nayak reviewed the patches, suggesting renames (`loose_object_map_insert()`, `loose_object_map_merge()`) and an error-message tweak; Justin Tobler declined the renames as not worth a reroll and plans to leave the series as-is.

## The day in brief
Harald Nordgren posted v13 of the shallow-repo fetch series with a new `@{push}` resolution commit, and v5 of the `--delete-integrated` branch series. Julia Evans's new-tutorial series reached v5 with Junio C Hamano supplying fixups for dangling `gittutorial` references. Elia Pinto posted two size_t-hardening series covering loose objects, notes, diff, and tree parsing on LLP64 platforms.

## Notable threads

### builtin/grep: read submodule trees via their repository
Patrick Steinhardt posted a three-patch series fixing `git grep --recurse-submodules` failing with "fatal: unable to read tree" on nested submodule trees, based on v2.56.0 for backporting. The first patch makes `grep_tree()` read nested submodule trees via `opt->repo` instead of `the_repository`; the other two read sparse directory entries and `.gitmodules` via the passed-in repository. Toon Claes reviewed, asking whether it should be noted as a 2.56 regression and suggesting `test_when_finished` cleanup, and said all changes look good.

### loose object size checks and LLP64 size truncations
Elia Pinto posted a six-patch series from a search for truncated sizes on LLP64 (64-bit Windows): uninitialized memory exposure from short loose-object zlib streams, missing size checks in the streaming loose-object reader, out-of-bounds writes in `git cat-file --batch` for 2-4GiB objects, heap overflow concatenating 4GiB+ notes, and out-of-bounds reads diffing 2-4GiB text files. The patches switch the loose-object reading path, `combine_notes_concatenate()`, and `count_trailing_blank()` to `size_t`.

### log: speed up --follow following renames for non-linear history
Miklos Vajna posted v4 of the series speeding up `git log --follow` across renames in non-linear history. The `-m --follow` fix is now a separate patch (3/6) so the simplify patch no longer changes behavior -- both bugs shared one cause, the followed path not being restored between merge parents -- and patch 6/6 fixes lost history when one merge side renamed the path while the dropped side edited the old name, keeping the linux.git case at about 37s.

### notes: do not drop notes written by a concurrent process
Farid Zakaria's patch makes `git notes` commands remember the commit they read and use it as the expected old ref value, so a concurrent update fails with "cannot lock ref" instead of silently losing notes. Kristoffer Haugsbakk argued against a three-way merge resolution for concurrent note updates. Farid Zakaria also stated the thread should be considered his GenAI disclosure and asked where the disclosure requirement is documented.

### tree-walk: use size_t for tree buffer sizes
Elia Pinto posted a two-patch series fixing truncation of tree objects of 4GiB or more: `struct tree_desc`'s `unsigned int` size field truncates on all platforms, silently dropping the rest of a tree when the cut falls between entries, affecting `git ls-tree` and `fsck_tree()`. Patch 1/2 moves `tree_desc` and its helpers to `size_t`; patch 2/2 does the same for `struct tree` and `parse_tree_buffer()` on Windows.

## In brief
- Kristofer Karlsson reported a preliminary v2 deducing the real object directory as the parent of the quarantine directory; Jeff King explained `GIT_QUARANTINE_PATH` is a path to support uses like GitHub's large-object rejection.
- Emily Shaffer clarified the voting process envisions N competing proposals with ranked-choice systems like CIVS, agreed votes should be secret, and accepted that eligibility rules might exclude her.
- Johannes Schindelin posted `Documentation/gitsecurityboundary.adoc` defining Git's security boundary around unexpected execution of attacker-controlled code, wired into `git.adoc`, `SECURITY.md`, and `command-list.txt`.
- Kaartic Sivaraam reviewed v3 of Christian Couder's early-scan parse-options series, leaving minor comments on `lookup_long_opt()` and `early_scan_options()` and calling the series much improved.
- Junio C Hamano identified a flaw in v4 2/4 of the imap-send header fix: `has_required_headers` detects end-of-headers by two consecutive LFs, but IMAP uses CRLF.
- Elia Pinto posted v2 of the size_t series for object and buffer sizes; Patrick Steinhardt reviewed the new 5GiB write test and patch 4, asking for a human touch in the commit message.
- Matt Hunter suggested folding a new helper into `get_follow_remote_head()`; Junio C Hamano questioned the reasoning, noting another topic wants the logic twice.
- Jasper Mayone posted the worktree series making `git worktree remove` refuse mid-rebase/bisect worktrees and `%(worktreepath)` name them; Junio C Hamano asked it be labeled v3, and Jasper agreed the next round will be v4.
- Stanislav Aleksandrov confirmed `^/` submodule URL semantics: absolute from the server root for a submodule that stays put when the superproject is forked.
- Harald Nordgren reported the ccache `pch_defines,time_macros` change merged into Meson and left Meson out of his patch.
- Christian Noé Ramos López posted v3 of the gpg doc clarifying SSH key lifetimes are checked against the committer/tagger timestamp; Patrick Steinhardt said it looks good.
- Omri Sarig posted v5 of the git-rebase doc fixing INCOMPATIBLE OPTIONS links; Tuomas Ahola noted a new typo and otherwise approved.
