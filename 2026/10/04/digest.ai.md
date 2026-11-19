# Git mailing list daily digest for 2026/10/04

## Long threads

### format-patch range-diff notes options

*Long thread (root 2026/08/24/20-35-41): 25 replies so far from 4 participants over 42 days; 4 messages that day.*

Kristoffer Haugsbakk's series teaches `git format-patch` the `--[no-]range-diff-notes` options so the range diff can show a different notes ref (or none) than the patches themselves, motivated by wanting to turn off range-diff notes while implementing the behavior in full generality. Today he posted v4, which trims expository text, refactors `rdiff_notes_cb` to call `parse_opt_string_list` instead of inlining it, and shortens the new options' documentation. Junio C Hamano reviewed v4 2/2, proposed a shorter replacement paragraph for the "why not error out like `--creation-factor`" explanation, and said the patch otherwise looks good; Kristoffer Haugsbakk agreed to use the proposed wording pretty much verbatim and posted v5, in which patch 2/2's commit message replaces the long paragraph with Junio's shorter wording and drops the thematic breaks, with patch 1/2 unchanged from v4.

### fetch behavior for new remotes in shallow repositories

*Long thread (root 2026/09/19/14-47-26): 23 replies so far from 4 participants over 16 days; 3 messages that day.*

Harald Nordgren's four-patch series avoids fetching every branch of a new remote in a shallow repository by adding `remote.<name>.refmap` as a per-remote default for `--refmap`, inferring branches to fetch from local upstreams when a remote has only a refmap configured, adding `git remote add --limited-fetch`, and defaulting to `--limited-fetch` in shallow repositories. Today he posted v6, whose only change from v5 is removing a leftover reference to the deleted default-branch logic in patch 3's commit message. Junio C Hamano reviewed v6, saying it is becoming much better, but identified a glitch: with only patch 1/4 applied, a remote configured with `remote.origin.refmap` but no tracked local branches makes `git fetch origin` die with "fatal: --refmap option is only meaningful with command-line refspec", which he called suboptimal when triggered by a configuration variable, and he raised an unresolved design question about what should happen when both `remote.origin.fetch` and `remote.origin.refmap` are configured. Harald Nordgren answered by proposing that `remote.<name>.fetch` should win in that case.

### CI failure and leak annotations linked to test scripts

*Long thread (root 2026/09/25/18-54-03): 20 replies so far from 4 participants over 10 days; 2 messages that day.*

Harald Nordgren's two-patch series links failure and leak annotations in GitHub CI to the test script so both can be found from the job summary: patch 1/2 adds `--immediate` to leak-sanitizer jobs and gives leaks their own annotation, and patch 2/2 points failure and fixed-known-breakage annotations at the test file and line. Today Harald Nordgren asked whether the inability to link annotations to test files unchanged in the PR is enough to call the change a regression, suggesting that part might not be worth doing; he objected to the tone of Phillip Wood's earlier message, acknowledged he should clarify that the change affects the summary page, and stated he had already commented on the regression issue, proposing to decide on that before progressing and noting the point of the topic was to make leak reporting less bad with everything else a bonus.

## The day in brief

Kristoffer Haugsbakk's `format-patch --[no-]range-diff-notes` series reached v5 after Junio C Hamano's wording suggestions, and Harald Nordgren's shallow-repository fetch series reached v6 with a design question about `remote.<name>.fetch` versus `remote.<name>.refmap` answered. Coy Geek reported that `git repack --drop-filtered --dry-run` on Git 2.56.0 writes promisor packs and honors `-d`, contradicting the documented no-write promise.

## Notable threads

### repack --drop-filtered --dry-run writes packs in Git 2.56.0

Coy Geek reported a bug in `builtin/repack.c` behavior on Git 2.56.0: `git repack -a --filter=blob:limit=1m --drop-filtered --dry-run` prints the candidate blob but also writes four new files (`.pack`, `.idx`, `.promisor`, `.rev`) to the pack directory, and adding explicit `-d` also deletes 12 existing pack and sidecar files, contradicting the manual's promise that `--dry-run` lists candidates without rebuilding packs or deleting anything. Both variants exit 0, print the candidate blob ID, and leave the tested blob available, so the report does not demonstrate loss of object content.

### fsmonitor untracked cache after a trivial response

Ilia K posted a patch to the fsmonitor daemon client code fixing a stale untracked-cache interaction: when the monitor sends a trivial response ("/"), `refresh_fsmonitor()` marks every index entry dirty but only clears the in-memory `use_fsmonitor` flag for the untracked cache, saving stale valid bits that later `git status` runs trust. The patch adds `untracked_cache_invalidate_stale_dirs()` to invalidate untracked-cache directories whose stat data changed or disappeared before the index is saved next to a new fsmonitor token, with a regression test covering a directory removed while no daemon is running.

### post-worktree lifecycle hook with prune notification

Domen Kožar posted v3 of a series adding a single `post-worktree` hook with four arguments (event, worktree identifier, old absolute path, new absolute path) for `add`, `move`, and `remove` operations, consolidating previous separate hooks; the motivating use case is devenv provisioning for worktrees created by an IDE or agent. Patch 2/2 extends the hook to pruning, emitting a `remove` event for each pruned administrative entry (none for `--dry-run`), modifying `should_prune_worktree()` to return the recorded `.git` path even when it points to a missing location, and continuing to process remaining entries after a hook failure while reflecting failures in the exit status.

### deferred fetch.followRemoteHEAD validation

Colin Hinton posted v5 of a patch to `builtin/fetch.c` deferring validation of the `fetch.followRemoteHEAD` configuration: instead of warning during config parsing even when the fetch never consults it, the raw config string is stored together with a `follow_remote_head_seen` flag and resolved lazily in `do_fetch()` only when the fetch would consult it, so a mistyped value warns and a missing value dies only then. The patch leaves NEEDSWORK comments at the still-unresolved `remote.<name>.followRemoteHEAD` handling in `remote.c`. Earlier today Junio C Hamano endorsed the `follow_remote_head_seen` approach and agreed that an empty string and a typo should behave the same, with `die()` now acceptable since validity is checked only when the value is consulted.

### --delete-merged detecting squash-merged branches

Harald Nordgren's patch teaches `git branch --delete-merged` to treat a branch as merged when some upstream commit since the fork point contains all of its changes, naming that commit in the output ("Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)"), which covers branches squash- or rebase-merged on GitHub whose tips are unreachable from upstream. Phillip Wood endorsed the approach but cautioned that the current implementation makes a separate upstream revision walk per branch and recalculates upstream diffs each time, suggesting the `git cherry` / `--cherry-mark` patch-id algorithm as a more efficient model and a tree/merge-base comparison for repositories with only non-rebased squash merges. Harald Nordgren agreed the per-branch walk can be improved, said he is iterating on sharing logic between branches, and asked whether an option to turn off patch-id based detection would still be wanted if there are no performance concerns.

## In brief

- Junio C Hamano withdrew his suggestion to use Message-IDs as link text for lore archive URLs in the gitbreaking-changes manpage after testing showed Asciidoctor renders a `mailto:` href that overrides the intended HTTP link; only the plain HTTP URL link remains.
- Harald Nordgren reposted v2 of "branch: let --delete-merged default to every upstream", naming the default pattern `**`; Phillip Wood objected that `PARSE_OPT_LASTARG_DEFAULT` is a usability footgun, and after an exchange about tone Harald Nordgren accepted the point as fair.
- Fionn Fitzmaurice clarified his Zsh completion patch excluding previous file arguments: word[1] is the subcommand so indexing from 2 skips it regardless of position, and he reported correct behavior after months of dogfooding; Junio C Hamano raised an edge case about a file named `add` in the working tree.
- Phillip Wood agreed that Alejandro Colomar's `git-rebase-walk` script for iterative rebases is best integrated into `git-rebase(1)` as an `--incremental` or `--progressive` option, preferring an implementation where `rebase --continue` handles progressively more recent bases.
- Junio C Hamano asked what `--no-...` means in the quoted `t/t9902-completion.sh` snippet of the "history: sign rewritten commits" series, specifically whether it is a literal completion candidate with three periods.
- Junio C Hamano judged Julia Evans's v2 doc-lint patch (no required SYNOPSIS in section 7 man pages) good enough, suggesting a more robust regex fallback to avoid an uninitialized-value warning.
- Junio C Hamano marked Kristoffer Haugsbakk's "doc: interpret-trailers: fix cmd examples" topic for `next`.
- Phillip Wood submitted a patch replacing the hard-coded integer `2` with the named constant `INCLUDE_ALL_FILES` in the `--all` option parsing for both `push_stash` and `save_stash` in `builtin/stash.c`.
