# Git mailing list daily digest for 2026/08/24

## Long threads

### `git history squash` subcommand for folding commit ranges

*Long thread (root 2026/06/14/19-25-39): 88 replies so far from 9 participants over 72 days; 2 messages that day.*

Harald Nordgren's series adds `git history squash <revision-range>` to `builtin/history.c`, folding a range of commits into one while preserving the oldest commit's authorship and parents, replaying descendants, resolving `fixup!`/`squash!`/`amend!` markers, and refusing when a local branch points into the range. Today Phillip Wood agreed that mixed-case OIDs have no compelling justification while accepting all-uppercase or all-lowercase OIDs seems reasonable, and asked whether uppercase OIDs were planned to be banned entirely as brian suggested, explaining the aim is to accept any OID that rebase would accept but not ref names. Junio C Hamano pointed to brian's `bc/restrict-hex-to-lowercase` topic and argued that, because Git never emitted uppercase hexadecimal itself, `starts_with()` would be sufficient to accept the OIDs users could have received after `fixup!` while still rejecting ref names.

### `worktree add` message for ambiguous remote branch names

*Long thread (root 2026/08/08/08-21-41): 28 replies so far from 6 participants over 17 days; 1 message that day.*

Yoichi Nakayama's series improves the error and advice shown when `git worktree add ../foo-dir bar-topic` matches multiple remote branches named `bar-topic`, listing the matched remotes in both `builtin/checkout.c` and `builtin/worktree.c`. Earlier, Junio C Hamano had classified the multiple-match-but-success case as the same class of bug being fixed and noted the existing behavior of creating the branch from HEAD on multiple matches is buggy and should eventually error out. Today Yoichi Nakayama agreed that treating multiple matches as an error is appropriate and stated he will make another commit to implement that previously postponed fix.

### packfile recovery when a multi-pack-index names a removed pack

*Long thread (root 2026/08/18/22-34-04): 16 replies so far from 5 participants over 7 days; 10 messages that day.*

Elijah Newren's two-patch series addresses objects being treated as missing despite being present, due to a race between geometric repacking and multi-pack-index updates: `git replay` SIGSEGVs on an unreadable merge input, and `find_pack_entry()` in `odb/source-packed.c` fails to find objects whose recorded owner pack was removed by a concurrent repack. Today Jeff King argued that non-QUICK callers are unaffected because the second read reloads packfiles, that repreparing packs on a QUICK miss would reintroduce the performance problems `OBJECT_INFO_QUICK` avoids, and that the mmapped-idx/missing-pack variant is fine because `packfile_fill_entry()` calls `is_pack_valid()`; he also proposed a tri-state return from `fill_midx_entry()` so the fallback runs only for known racy negatives. Patrick Steinhardt countered that the patch fixes a real bug regardless of QUICK: the non-MIDX lookup skips all MIDX-covered packs, hiding secondary copies when the preferred pack vanishes. Derrick Stolee explained that `git multi-pack-index expire` exists for exactly this race, keeping stale packs in the MIDX to halt reads by new processes while allowing existing processes to read them, and suggested retaining redundant data for a time interval to reduce how often the fallback triggers.

## The day in brief
Karthik Nayak posted v3 of the report hook for `git-receive-pack`, with a non-zero hook exit now rejecting references with a "report hook failed" error instead of dying, while Junio C Hamano and Patrick Steinhardt objected to the generic name "report" and proposed "receive-report". Karthik Nayak also posted v2 of the reftable stack series avoiding reloads while locked, with corrected benchmarks showing a consistent 15-20% improvement. Elijah Newren's multi-pack-index race fix drew a substantive exchange between Jeff King, Patrick Steinhardt, and Derrick Stolee over when the fallback search should run.

## Notable threads

### report hook for `git-receive-pack`
Karthik Nayak's series introduces a report hook for `git-receive-pack(1)`, with documentation in `git-receive-pack.adoc` and response generation moved into a new `generate_response()` function in receive-pack. Today he posted v3, in which a non-zero hook exit no longer die()s but modifies every reference to be rejected with a "report hook failed" error, and the docs link to `linkgit:gitprotocol-pack[5]`. Patrick Steinhardt asked whether each reference line in the pkt-line status report should end with a newline, and Karthik Nayak said he will clarify that in the documentation. Junio C Hamano objected that "report" is far too generic a name for the hook, and Patrick Steinhardt agreed, proposing "receive-report" or something similar.

### reftable stack: avoid reloading when locked
Karthik Nayak's series addresses the reftable library reloading the stack on every call inside a reference transaction, where the stack is already locked and reloading is a no-op. Today he posted v2 of the four-patch series, adding a `locked` bit field to `reftable_addition` so multiple additions on the same stack do not release each other's list lock, moving the list-file lock into `struct reftable_stack`, making `stack_uptodate()` skip reloading when already locked, and correcting the benchmarks to use the `refs/tags/` namespace, showing a consistent 15-20% improvement with `newfstatat()` calls constant instead of growing linearly. Jeff King reviewed patch 4/4, calling the corrected benchmark results "much nicer" and stating the idea is sound.

### `git config --global` reading both home and XDG files
Delilah's series fixes the inconsistency where `git config list --global` reads only `$HOME/.gitconfig` while plain `git config list` reads both it and `$XDG_CONFIG_HOME/git/config`. Today Junio C Hamano explained that reading per-user configuration from both locations was a deliberate design choice to help users migrate away from `~/.gitconfig`, that `git config --global --edit` should follow the same choice as the write-out path, but acknowledged that the `--global` read path was not designed as carefully and that any discrepancy between what `git config` and `git config --global` read from per-user files is very likely a bug caused by a slip of the mind.

### sequencer releasing the ODB before spawning `git commit`
Johannes Schindelin's patch releases the object database's file handles and memory mappings before the sequencer spawns `git commit`, so that auto-maintenance triggered by the child can repack on Windows without unlink retry prompts. Today Phillip Wood said the patch looks fine as an immediate fix but suggested passing `-c gc.auto=false` when the sequencer forks `git commit`, and proposed studying whether the ODB could be automatically closed when forking another git command. Junio C Hamano endorsed studying that feasibility so fixes like this would not need to be sprinkled across many code paths.

### `format-patch --[no-]range-diff-notes`
Kristoffer Haugsbakk's series teaches `format-patch` options to tweak notes output in the range diff independently of the notes in the patches. Today he implemented `--[no-]range-diff-notes[=<ref>]`, simplified `get_notes_args` to take only `struct rev_info`, and renamed the `rdiff_log_arg` member to `rdiff_notes_arg`. Junio C Hamano questioned whether `--no-notes --range-diff-notes` would display notes in the range-diff, noting that if the bare option falls back to the patches' notes behavior, the range-diff would not include them.

## In brief
- Artur Bieniek clarified that his `git pull --hard` proposal is about updating the checked-out working tree, and proposed the inverse `git reset --pull --hard`; Junio C Hamano suggested an alias might be the appropriate feature for this use case.
- Patrick Steinhardt objected to Nikolaus Schuetz's tests pinning symbolic-ref exit codes 1 vs 128, and Junio C Hamano agreed not checking the exact code avoids overspecifying; Phillip Wood countered the difference may be intentional for scripts, and Schuetz proposed corrected `git-symbolic-ref.adoc` wording.
- Patrick Steinhardt concurred with Elijah Newren's shallow-clone push problem that the proper fix is enabling `push.negotiate` by default, and proposed a `push.negotiate=shallow` option enabled automatically for shallow clones.
- Phillip Wood proposed an alternative to Vladimir Sitnikov's configurable stash ref: an option to pop the last stash created on the current branch; Junio C Hamano cautioned against relying heavily on stash messages.
- Karthik Nayak posted v2 1/4 removing the `REFTABLE_STACK_NEW_ADDITION_RELOAD` flag, making stack reloading the default flow when a locked stack is outdated.
- Karthik Nayak posted v2 2/4 renaming `reftable_stack_new_addition()` to `reftable_stack_addition_new()`.
- Oswald Buddenhagen preferred `lib/odb/` over `odb/` and objected to double-quoted includes as implementation-defined; Junio C Hamano disagreed with the `lib/odb/` suggestion and asked for clarification on include style.
- Jeff King explained `--ext-diff` exists because external diffs can perform arbitrary actions like opening GUI windows, making it surprising for `git log` to open dozens while walking history, and suggested textconv as often a better fit.
- Weijie Yuan raised a DCO provenance question about AI-assisted patches on the `utime.h` to `utimensat` migration series; Junio C Hamano explained "sufficient due diligence" means checking code against the project's own past fixes, and Oswald Buddenhagen argued such diligence cannot be assumed when an AI tool is used.
- Weijie Yuan acknowledged he understood the provenance explanation and apologized for bringing Dscho into the discussion.
- Patrick Steinhardt confirmed several topics in the "What's cooking" report can be merged or are ready, including `ps/odb-geometric-repack-loose-threshold`, `fr/pack-objects-trace-pack-bytes`, `ps/odb-pluggable-pack-generation`, and `jt/receive-pack-pluggable-writes`; Junio C Hamano said he is happy with the `dk/use-nsec-runtime` iteration modulo a log-message nit.
- Junio C Hamano posted a typofix adding a missing preposition in `Documentation/technical/rerere.adoc`, and Michael Montalbo responded "LGTM".
