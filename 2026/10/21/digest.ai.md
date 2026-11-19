# Git mailing list daily digest for 2026/10/21

## Long threads

### branch: detect squash- and rebase-merged branches in `--delete-merged`

*Long thread (root 2026/09/29/07-30-30): 35 replies so far from 6 participants over 23 days; 3 messages that day.*

Harald Nordgren posted v3 of the series that makes `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains the branch's version of every file it changed, comparing object IDs only. The v2 `--no-squashed` opt-out became an opt-in `--squashed` flag that also catches squash/rebase merges where upstream changed the same files, by merging the branch's version into upstream's and counting it landed when upstream is left unchanged; the series is split into three commits, with the `branch.<name>.deleteMerged=false` opt-out moved into a helper honored by the new path. Jasper Mayone ran v3 on generated repositories: tests pass and `--squashed` deletes the new case with the correct "landed as" commit, but the v2 false positive remains -- a never-merged branch is deleted because another author made the same change in an unrelated commit, which v3's default cheap check triggers with no way to disable; he prefers the looser check behind one opt-in with a config default. Harald Nordgren replied that such a prune would feel weird but called it an extremely rare edge case, noted the content is already upstream even if pruned, and rejected the single opt-in plus config-variable design as too much UX mental load.

### connected: incremental connectivity check via rev-list

*Long thread (root 2026/09/14/09-47-54): 29 replies so far from 3 participants over 38 days; 2 messages that day.*

Kristofer Karlsson posted v4 (0/6) of the series adding an incremental mode for the connectivity check in `connected.c`, gated behind `transfer.connectivityCheck=incremental` or `"auto"`, to keep the check from slowing as the reachable-object boundary grows. The verifier runs inside the rev-list subprocess via a new `--connectivity-check=` option; v4 adds the "auto" mode with a configurable cutoff (default 1000 commits, `auto=<cutoff>`) and falls back to "full" for unknown config values, clones, pushes into empty repositories, and deepening fetches. The series also adds `Documentation/technical/connectivity-check.adoc`, p5413 perf tests, a refactor dropping `connected.c`'s dependency on the global repository, and `tree-verify.c` with `verify_commits_incremental()`. Junio C Hamano reviewed patch 5/6, questioning whether assuming all non-full modes want to avoid lazy fetching is a trap for future developers and whether `verify_blob` needs an extended version with explicit flag bits.

### doc: new beginner Git tutorial

*Long thread (root 2026/10/06/19-37-00): 21 replies so far from 5 participants over 16 days; 2 messages that day.*

Julia Evans's v2 series replaces the old `gittutorial` with a new 739-line `Documentation/gittutorial.adoc` for beginners. Today's exchange concerned heading style: Junio C Hamano corrected his earlier wording to note that mistakes are "easier to correct with the ATX-style prefix format," while stating he has no strong preference either way. Julia Evans said she finds the prefix format easier for consistency, but formatting `DESCRIPTION` as `== DESCRIPTION` makes `make lint-docs` report the required section missing, so she prefers keeping the usual heading format rather than mixing formats or updating the linter.

### doc: move BreakingChanges to a manpage

*Long thread (root 2026/09/28/10-41-24): 20 replies so far from 5 participants over 24 days; 3 messages that day.*

Kristoffer Haugsbakk's series moves `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage for visibility via `git help`. Junio C Hamano raised the question of what to do when Git 3.0 arrives -- whether every section would need rewriting to say "we used to do this differently before 3.0". Kristoffer Haugsbakk suggested a trailing section for "what happened in 3.0", introduced with text noting the changes are written in the future tense for readers coming from version 2.98. Junio C Hamano responded that the rewriting effort is not the major concern; the real question is whether keeping such a historical document is worth it, asking whether anyone still cares about the Git 1.x-to-2.0 transition issues.

### worktree: copy-on-write creation and ephemeral branches

*Long thread (root 2026/06/05/18-49-26): 11 replies so far from 5 participants over 139 days; 9 messages that day.*

Jason Newton posted v3 (0/4) of the series adding `git worktree add --reflink` (reflinking files and index on copy-on-write filesystems, falling back to a normal checkout; disabled on Windows because of the ReFS zero-fill bug) and reaping of ephemeral branches auto-created by `worktree add` when the worktree is removed. Eric Sunshine objected that the ephemeral-branch patch adds complexity for a use case already covered by `--detach`, and asked why `--reflink` must stay opt-in; Jason Newton argued a detached HEAD leaves status/push/rename with nothing to name while the worktree exists. Eric Sunshine criticized the reply as an LLM-generated response, and Phillip Wood said the message fell short of the standard of constructive discussion; Jason Newton defended mixed human/LLM authorship and said worktrees behaving like branches is the common user expectation.

## The day in brief

Harald Nordgren's v3 rework of `git branch --delete-merged` to detect squash- and rebase-merged branches drew fresh testing and a debate over a false positive that deletes never-merged branches. Kristofer Karlsson posted v4 of the incremental connectivity check with an "auto" mode, and Masaya Kazama reported that a stale `maintenance.lock` silently disables auto-maintenance. Junio C Hamano queued Phillip Wood's stash-creation cleanup series.

## Notable threads

### stash: avoid unnecessary index loads and refreshes

Junio C Hamano reviewed Phillip Wood's v3 series, which removes redundant `repo_read_preload_index()` and `refresh_index()` calls from `git stash push`/`create`, calling the cover letter "nicely split and described" and "quite well done" and stating "Will queue." On patch 1/7 he suggested defining `CREATE_SUCCESS = 0` explicitly and using it instead of bare `0` in the return-variable initializations, while noting it was not worth a reroll.

### remote.c: defer remote.<name>.followRemoteHEAD validation

Colin Hinton said he likely will revert the addition of "warn-if-not-<branch>" support for `fetch.followRemoteHEAD`, restricting it to `remote.<name>.followRemoteHEAD` only, and will revert the corresponding `.adoc` changes. He agreed with Junio C Hamano that the error messages should name the `fetch.` prefix, committed to cleaning up overly-long lines, and agreed to experiment with moving the `follow_remote_head_settings` enum definition to the fetch file in v2.

### odb/source-files: reload alternates on retry

Patrick Steinhardt agreed with Junio C Hamano that patch 4/4 updates the stat validity before the alternates file is successfully read, and proposed only calling `stat_validity_update()` after a successful read and parse, clearing validity on failure instead. Christian Couder pointed out that patch 2/4's `prepare_alternates()` reads alternates from `files->dirs`, which is the temporary quarantine directory during receive-pack, so no alternates are found there; he suggested always reloading the repository's real alternates.

### maintenance: stale maintenance.lock silently disables auto-maintenance

Masaya Kazama reported that since 6e95b07e5f the detached `git maintenance` child holds `objects/maintenance.lock` for the whole background run, so a SIGKILL or power loss leaves a stale lock that makes every later `git maintenance run --auto` return silently without running. He provided a reproduction and described real-world impact: in a large partial clone a stale lock let packs grow to 13.9 GB until a manual run reduced them to 0.68 GB. He listed three unimplemented options: leave a user-findable trace when `--auto` skips, record the actual holder's pid and hostname, or use an OS-level lock released on process death.

### refs/files: keep root refs out of loose ref cache

Junio C Hamano marked the topic for `next` after Patrick Steinhardt's review approval. The patch keeps root refs such as `HEAD` out of the loose ref cache in `refs/files-backend.c`, reading them into a separate temporary cache only when an iterator requests `REFS_FOR_EACH_INCLUDE_ROOT_REFS`, fixing `git for-each-ref --include-root-refs --merged` silently omitting root refs.

## In brief

- Andrew Parmet reported a partial-clone bug where a merge fails with "not our ref" because it requests a locally created blob from the promisor remote, with a minimal reproducer.
- Christian Noé Ramos López posted a patch making signature verification fail with an error when `gpg.ssh.revocationFile` names a missing file, supporting `:(optional)` to skip the check; Junio C Hamano reviewed and suggested folding the check into the earlier failure branch.
- Patrick Steinhardt endorsed advertising uncommitted fetched tips as "haves" during tag backfill instead of committing the intermediate ref transaction; Justin Tobler and Karthik Nayak debated whether that is conceptually sound.
- Phillip Wood reviewed Erik Cervin-Edin's v3 allowing `-m`/`-F`/`-c`/`-C` with all `--fixup` variations, asking for tests that the editor is or is not invoked.
- Justin Tobler posted v4 fixing ".keep" lockfile ownership in ODB transactions, with `ret` now set on all exit paths in `write_pack_lockfile()`.
- **v2 checkout tracking fix series**: tsukasa inoue posted v2 making `git switch -c` and `checkout -b` finish the switch and exit 0 when the tracking config cannot be written. Junio C Hamano replied "OK" to the series, then asked whether 2/3 and 3/3 should be a separate topic from the hint fix.

- Patrick Steinhardt posted v2 of the `extensions.objectStorage` series (currently only the "files" backend); Karthik Nayak reviewed and is okay with the series as is.
- Christian Couder posted a 13-patch series adding pack index v3 for SHA-1/SHA-256 interoperability; Junio C Hamano noted the DCO requires Signed-off-by on relayed patches.
- Elia Pinto posted a 3-patch series converting object and buffer sizes to `size_t` to fix silent truncation above 4 GiB on 64-bit Windows; Patrick Steinhardt called the odb write-side bug serious and asked for an EXPENSIVE test.
- Nakul V S posted a patch refusing to revert shallow boundary commits by default, with `--allow-shallow-boundary` to override; Phillip Wood and Junio C Hamano asked whether cherry-pick and rebase should be covered too.
- Emily Shaffer posted `Documentation/CommunityVoting.adoc` outlining a formal voting process; Junio C Hamano endorsed reserving votes for policy decisions and suggested identifying ballot proposals by object name.
