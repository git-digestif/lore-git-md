# Git mailing list daily digest for 2026/08/24

## Long threads

### `git history squash` subcommand for folding a commit range

*Long thread: 88 replies so far from 9 participants over 72 days; 2 messages that day.*

The series adds `git history squash <revision-range>` in `builtin/history.c`, folding a range of commits into one while preserving the oldest commit's authorship and parents, replaying descendants, and refusing when a local branch points into the range. Earlier revisions split the implementation into five patches and reworked range validation and autosquash-marker resolution. Today Phillip Wood agreed that mixed-case OIDs have no compelling justification, accepting all-uppercase or all-lowercase forms, and asked whether uppercase OIDs were planned to be banned entirely; Junio C Hamano pointed to brian's `bc/restrict-hex-to-lowercase` topic and argued that, since Git never emitted uppercase hexadecimal itself, `starts_with()` would accept the OIDs users could have received after `fixup!` while still rejecting ref names.

### worktree add message for ambiguous remote branch names

*Long thread: 28 replies so far from 6 participants over 17 days; 1 message that day.*

The series improves the error and advice shown by `git worktree add` and `git checkout` when a branch name matches multiple remote-tracking branches, listing the matched remotes in both commands. Earlier revisions split the work into three commits and reworded the advice to "Branch name '%s' appears in multiple remotes:". Today Yoichi Nakayama agreed with Junio C Hamano that treating multiple matches as an error is appropriate and said he will make another commit implementing that previously postponed fix.

### packfile recovery when a multi-pack-index names a removed pack

*Long thread: 16 replies so far from 5 participants over 7 days; 10 messages that day.*

The two-patch series fixes objects being treated as missing when a geometric repack removes a pack that an old multi-pack-index still names, teaching `find_pack_entry()` in `odb/source-packed.c` to scan the MIDX's covered packs for a surviving copy, and fixing a `git replay` SIGSEGV on unreadable merge input. Today Jeff King argued that non-QUICK callers are unaffected because the second read reloads packfiles, and that repreparing packs on a QUICK miss would reintroduce the performance problems `OBJECT_INFO_QUICK` avoids; Patrick Steinhardt countered that the patch fixes a real bug because the non-MIDX lookup skips all MIDX-covered packs, hiding secondary copies regardless of QUICK. Derrick Stolee explained that `git multi-pack-index expire` exists for exactly this race and suggested retaining redundant packs for a time interval to reduce fallback triggers.

## The day in brief
The `git history squash` series drew a fresh round of review on how `fixup!` subject matching should treat hexadecimal case. The worktree ambiguous-remote-branch series gained agreement that multiple matches should become an error. And the multi-pack-index removed-pack race sparked a detailed exchange about `OBJECT_INFO_QUICK` semantics and fallback lookups.

## Notable threads

### report hook for git-receive-pack
Karthik Nayak posted v3 of the series introducing a report hook for `git-receive-pack(1)`, where a non-zero hook exit now rejects each reference with "report hook failed" instead of dying. Junio C Hamano objected that "report" is far too generic a name for the hook; Patrick Steinhardt agreed and proposed "receive-report" or something similar.

### reftable stack reload avoidance when locked
Karthik Nayak posted v2 of the series avoiding stack reloads inside a locked reference transaction in the reftable library, adding a `locked` bit to `reftable_addition` so additions on the same stack do not release each other's lock, with benchmarks showing a consistent 15-20% improvement and constant `newfstatat()` counts. Jeff King reviewed patch 4/4, calling the corrected benchmarks "much nicer" and the idea sound.

### config --global reading both home and XDG files
The series makes `git config --global` read both `$HOME/.gitconfig` and the XDG config file, matching what plain `git config` reads. Junio C Hamano explained that reading both per-user files was a deliberate migration aid, but acknowledged the `--global` read path was not designed as carefully and that discrepancies with the general sequence are very likely bugs.

## In brief
- Phillip Wood called the sequencer ODB-release patch a fine immediate fix but suggested `-c gc.auto=false` when forking `git commit`, and both he and Junio C Hamano endorsed studying automatic ODB closure when forking Git commands.
- Patrick Steinhardt proposed a `push.negotiate=shallow` option, enabled by default for shallow clones, instead of new send-pack logic for withholding the shallow boundary tree.
- Kristoffer Haugsbakk implemented `--[no-]range-diff-notes[=<ref>]` for `format-patch`; Junio C Hamano questioned whether `--no-notes --range-diff-notes` would still show notes in the range-diff.
- Reviewers disagreed over pinning `git symbolic-ref` exit codes 1 vs 128; Nikolaus Schuetz favored not testing the exact code and offered to sync the adoc NOTES.
- Phillip Wood proposed a per-branch stash pop option as an alternative to a configurable stash ref; Junio C Hamano cautioned against relying heavily on stash messages.
- Artur Bieniek proposed the inverse `git reset --pull --hard` design; Junio C Hamano suggested an alias may be the right feature for this use case.
- **Yuan on AI DCO; Buddenhagen on AI**: Weijie Yuan retracted earlier remarks and reframed the question around AI-assisted DCO risk; Oswald Buddenhagen argued due diligence cannot be assumed with AI tools.
